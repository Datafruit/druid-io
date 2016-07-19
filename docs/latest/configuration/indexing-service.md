---
layout: doc_page
---

对于一般索引服务信息，查阅[这里](../design/indexing-service.html)。
## Runtime Configuration 

索引服务使用的几个全球配置在[配置](../configuration/index.html)和具有以下配置：
### 必须设置Overlord和Middle Manager

#### 节点配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.host`|The host for the current node. This is used to advertise the current processes location as reachable from another node and should generally be specified such that `http://${druid.host}/` could actually talk to this process|InetAddress.getLocalHost().getCanonicalHostName()|
|`druid.port`|This is the port to actually listen on; unless port mapping is used, this will be the same port as is on `druid.host`|8090|
|`druid.service`|The name of the service. This is used as a dimension when emitting metrics and alerts to differentiate between the various services|druid/overlord|

#### 任务日志

如果你正在远程模式下运行索引服务，任务日志必须存储在S3，Azure Blob Store或者HDFS。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.logs.type`|Choices:noop, s3, azure, hdfs, file. Where to store task logs|file|

##### 文件任务日志

任务日志存储在本地文件系统。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.logs.directory`|Local filesystem path.|log|

##### S3任务日志

任务日志存储在S3。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.logs.s3Bucket`|S3 bucket name.|none|
|`druid.indexer.logs.s3Prefix`|S3 key prefix.|none|

#### Azure Blob Store任务日志
任务日志存储在Azure Blob Store。

注意：这个使用和Azure深存储模块相同的存储账户。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.logs.container`|The Azure Blob Store container to write logs to|none|
|`druid.indexer.logs.prefix`|The path to prepend to logs|none|

##### HDFS任务日志

任务日志存储在HDFS。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.logs.directory`|The directory to store logs.|none|

### Overlord 配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.runner.type`|Choices "local" or "remote". Indicates whether tasks should be run locally or in a distributed environment.|local|
|`druid.indexer.storage.type`|Choices are "local" or "metadata". Indicates whether incoming tasks should be stored locally (in heap) or in metadata storage. Storing incoming tasks in metadata storage allows for tasks to be resumed if the overlord should fail.|local|
|`druid.indexer.storage.recentlyFinishedThreshold`|A duration of time to store task results.|PT24H|
|`druid.indexer.queue.maxSize`|Maximum number of active tasks at one time.|Integer.MAX_VALUE|
|`druid.indexer.queue.startDelay`|Sleep this long before starting overlord queue management. This can be useful to give a cluster time to re-orient itself after e.g. a widespread network issue.|PT1M|
|`druid.indexer.queue.restartDelay`|Sleep this long when overlord queue management throws an exception before trying again.|PT30S|
|`druid.indexer.queue.storageSyncRate`|Sync overlord state this often with an underlying task persistence mechanism.|PT1M|

以下的配置仅应用于Overlord运行在远程模式时：
|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.runner.taskAssignmentTimeout`|How long to wait after a task as been assigned to a middle manager before throwing an error.|PT5M|
|`druid.indexer.runner.minWorkerVersion`|The minimum middle manager version to send tasks to. |"0"|
|`druid.indexer.runner.compressZnodes`|Indicates whether or not the overlord should expect middle managers to compress Znodes.|true|
|`druid.indexer.runner.maxZnodeBytes`|The maximum size Znode in bytes that can be created in Zookeeper.|524288|
|`druid.indexer.runner.taskCleanupTimeout`|How long to wait before failing a task after a middle manager is disconnected from Zookeeper.|PT15M|
|`druid.indexer.runner.taskShutdownLinkTimeout`|How long to wait on a shutdown request to a middle manager before timing out|PT1M|

这是自动缩放（如果是启动的）额外配置：
|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.autoscale.strategy`|Choices are "noop" or "ec2". Sets the strategy to run when autoscaling is required.|noop|
|`druid.indexer.autoscale.doAutoscale`|If set to "true" autoscaling will be enabled.|false|
|`druid.indexer.autoscale.provisionPeriod`|How often to check whether or not new middle managers should be added.|PT1M|
|`druid.indexer.autoscale.terminatePeriod`|How often to check when middle managers should be removed.|PT5M|
|`druid.indexer.autoscale.originTime`|The starting reference timestamp that the terminate period increments upon.|2012-01-01T00:55:00.000Z|
|`druid.indexer.autoscale.workerIdleTimeout`|How long can a worker be idle (not a run task) before it can be considered for termination.|PT90M|
|`druid.indexer.autoscale.maxScalingDuration`|How long the overlord will wait around for a middle manager to show up before giving up.|PT15M|
|`druid.indexer.autoscale.numEventsToTrack`|The number of autoscaling related events (node creation and termination) to track.|10|
|`druid.indexer.autoscale.pendingTaskTimeout`|How long a task can be in "pending" state before the overlord tries to scale up.|PT30S|
|`druid.indexer.autoscale.workerVersion`|If set, will only create nodes of set version during autoscaling. Overrides dynamic configuration. |null|
|`druid.indexer.autoscale.workerPort`|The port that middle managers will run on.|8080|

#### Dynamic 配置

Overlord可以动态地改变工作者的行为。
JSON对象可以在：
```
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker
```
通过一个post请求提交到Overlord

审计配置的可选头参数也可以被指定改变。
|头参数名|描述|默认|
|----------|-------------|---------|
|`X-Druid-Author`| author making the config change|""|
|`X-Druid-Comment`| comment describing the change being done|""|

样本工人配置规格如下： 
```json
{
  "selectStrategy": {
    "type": "fillCapacityWithAffinity",
    "affinityConfig": {
      "affinity": {
        "datasource1": ["host1:port", "host2:port"],
        "datasource2": ["host3:port"]
      }
    }
  },
  "autoScaler": {
    "type": "ec2",
    "minNumWorkers": 2,
    "maxNumWorkers": 12,
    "envConfig": {
      "availabilityZone": "us-east-1a",
      "nodeData": {
        "amiId": "${AMI}",
        "instanceType": "c3.8xlarge",
        "minInstances": 1,
        "maxInstances": 1,
        "securityGroupIds": ["${IDs}"],
        "keyName": ${KEY_NAME}
      },
      "userData": {
        "impl": "string",
        "data": "${SCRIPT_COMMAND}",
        "versionReplacementString": ":VERSION:",
        "version": null
      }
    }
  }
}
```


相同的URL发出一个GET请求将返回当前位置的人员配置规格。
上面人员配置规格列表仅是EC2的一个示例，它可以扩展其他部署环境的代码库。人员配置规格描述如下：

|属性|描述|默认|
|--------|-----------|-------|
|`selectStrategy`|How to assign tasks to middle managers. Choices are `fillCapacity`, `fillCapacityWithAffinity`, `equalDistribution` and `javascript`.|fillCapacity|
|`autoScaler`|Only used if autoscaling is enabled. See below.|null|

查看审计人员配置历史，发出GET请求到URL-
```
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker/history?interval=<interval>
```

默认的间隔值可以被指定，通过在Overlord Runtime.properties设置`druid.audit.manager.auditHistoryMillis`（如果没设置则是一周）。
查看最近n条审计员工配置历史，发出GET请求到URL-
```
http://<OVERLORD_IP>:<port>/druid/indexer/v1/worker/history?count=<n>
```

#### 员工选择策略

##### Fill Capacity

按照员工能力分配任务。
|属性|描述|默认|
|--------|-----------|-------|
|`type`|`fillCapacity`.|fillCapacity|

##### Fill Capacity With Affinity

一个affinity配置提供如下。
|属性|描述|默认|
|--------|-----------|-------|
|`type`|`fillCapacityWithAffinity`.|fillCapacityWithAffinity|
|`affinity`|JSON object mapping a datasource String name to a list of indexing service middle manager host:port String values. Druid doesn't perform DNS resolution, so the 'host' value must match what is configured on the middle manager and what the middle manager announces itself as (examine the Overlord logs to see what your middle manager announces itself as).|{}|

任务将先分配给首选的员工。如果没有首选指定的数据源则使用填补能力策略
##### 平均分配

员工大部分任务是被分配的。
|属性|描述|默认|
|--------|-----------|-------|
|`type`|`equalDistribution`.|fillCapacity|

##### Javascript

允许为选择的员工定义任意逻辑去用一个JavaScript函数执行任务。
函数传递remoteTaskRunnerConfig，workerId的映射使工人和任务可执行并返回workerId，如果不能运行任务则任务应该运行或null。
他可用于缺失功能的快速发展，工人选择逻辑是经常被改变或者转换的。
如果选择逻辑非常复杂，在javascript环境下不容易测试，最好用java写一个扩展当前员工选择策略的druid扩展模块。


|属性|描述|默认|
|--------|-----------|-------|
|`type`|`javascript`.|javascript|
|`function`|String representing javascript function||

例子：发送batch_index_task到员工10.0.0.1 和 10.0.0.2和所有其他的任务发送到其他可用的员工的一个函数。
```
{
"type":"javascript",
"function":"function (config, zkWorkers, task) {\nvar batch_workers = new java.util.ArrayList();\nbatch_workers.add(\"10.0.0.1\");\nbatch_workers.add(\"10.0.0.2\");\nworkers = zkWorkers.keySet().toArray();\nvar sortedWorkers = new Array()\n;for(var i = 0; i < workers.length; i++){\n sortedWorkers[i] = workers[i];\n}\nArray.prototype.sort.call(sortedWorkers,function(a, b){return zkWorkers.get(b).getCurrCapacityUsed() - zkWorkers.get(a).getCurrCapacityUsed();});\nvar minWorkerVer = config.getMinWorkerVersion();\nfor (var i = 0; i < sortedWorkers.length; i++) {\n var worker = sortedWorkers[i];\n  var zkWorker = zkWorkers.get(worker);\n  if(zkWorker.canRunTask(task) && zkWorker.isValidVersion(minWorkerVer)){\n    if(task.getType() == 'index_hadoop' && batch_workers.contains(worker)){\n      return worker;\n    } else {\n      if(task.getType() != 'index_hadoop' && !batch_workers.contains(worker)){\n        return worker;\n      }\n    }\n  }\n}\nreturn null;\n}"
}


```


#### Autoscaler

Amazon's EC2当前仅支持自动缩放。

|属性|描述|默认|
|--------|-----------|-------|
|`minNumWorkers`|The minimum number of workers that can be in the cluster at any given time.|0|
|`maxNumWorkers`|The maximum number of workers that can be in the cluster at any given time.|0|
|`availabilityZone`|What availability zone to run in.|none|
|`nodeData`|A JSON object that describes how to launch new nodes.|none; required|
|`userData`|A JSON object that describes how to configure new nodes. If you have set druid.indexer.autoscale.workerVersion, this must have a versionReplacementString. Otherwise, a versionReplacementString is not necessary.|none; optional|

### MiddleManager 配置

Middle Manager通过他们的配置下至到他们的子工人。Middle Manager要求以下配置：

|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.runner.allowedPrefixes`|Whitelist of prefixes for configs that can be passed down to child peons.|"com.metamx", "druid", "io.druid", "user.timezone","file.encoding"|
|`druid.indexer.runner.compressZnodes`|Indicates whether or not the middle managers should compress Znodes.|true|
|`druid.indexer.runner.classpath`|Java classpath for the peon.|System.getProperty("java.class.path")|
|`druid.indexer.runner.javaCommand`|Command required to execute java.|java|
|`druid.indexer.runner.javaOpts`|-X Java options to run the peon in its own JVM.|""|
|`druid.indexer.runner.maxZnodeBytes`|The maximum size Znode in bytes that can be created in Zookeeper.|524288|
|`druid.indexer.runner.startPort`|The port that peons begin running on.|8100|
|`druid.indexer.runner.separateIngestionEndpoint`|Use separate server and consequently separate jetty thread pool for ingesting events|false|
|`druid.worker.ip`|The IP of the worker.|localhost|
|`druid.worker.version`|Version identifier for the middle manager.|0|
|`druid.worker.capacity`|Maximum number of tasks the middle manager can accept.|Number of available processors - 1|


#### Peon 配置
虽然工人继承他们父母Middle Manager的配置，明确子工人配置在Middle Manage可以通过加上前缀设置:

```
druid.indexer.fork.property
```
额外的工人配置包括：
|属性|描述|默认|
|--------|-----------|-------|
|`druid.peon.mode`|Choices are "local" and "remote". Setting this to local means you intend to run the peon as a standalone node (Not recommended).|remote|
|`druid.indexer.task.baseDir`|Base temporary working directory.|`System.getProperty("java.io.tmpdir")`|
|`druid.indexer.task.baseTaskDir`|Base temporary working directory for tasks.|`${druid.indexer.task.baseDir}/persistent/tasks`|
|`druid.indexer.task.hadoopWorkingPath`|Temporary working directory for Hadoop tasks.|`/tmp/druid-indexing`|
|`druid.indexer.task.defaultRowFlushBoundary`|Highest row count before persisting to disk. Used for indexing generating tasks.|50000|
|`druid.indexer.task.defaultHadoopCoordinates`|Hadoop version to use with HadoopIndexTasks that do not request a particular version.|org.apache.hadoop:hadoop-client:2.3.0|
|`druid.indexer.task.restoreTasksOnRestart`|If true, middleManagers will attempt to stop tasks gracefully on shutdown and restore them on restart.|false|
|`druid.indexer.task.gracefulShutdownTimeout`|Wait this long on middleManager restart for restorable tasks to gracefully exit.|PT5M|
|`druid.indexer.task.directoryLockTimeout`|Wait this long for zombie peons to exit before giving up on their replacements.|PT10M|

如果`druid.indexer.runner.separateIngestionEndpoint`是设置为true那么下面的配置对工人摄取服务器是可行的：
|属性|描述|默认|
|--------|-----------|-------|
|`druid.indexer.server.chathandler.http.numThreads`|Number of threads for HTTP requests.|Math.max(10, (Number of available processors * 17) / 16 + 2) + 30|
|`druid.indexer.server.chathandler.http.maxIdleTime`|The Jetty max idle time for a connection.|PT5m|

如果工人在远程模式下运行，必须要有一个Overlord启动和运行。工人在远程模式可以设置以下配置：

|属性|描述|默认|
|--------|-----------|-------|
|`druid.peon.taskActionClient.retry.minWait`|The minimum retry time to communicate with overlord.|PT1M|
|`druid.peon.taskActionClient.retry.maxWait`|The maximum retry time to communicate with overlord.|PT10M|
|`druid.peon.taskActionClient.retry.maxRetryCount`|The maximum number of retries to communicate with overlord.|10|
