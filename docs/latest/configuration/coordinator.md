---
layout: doc_page
---
Coordinator 节点配置
==============================
一般的Coordination节点信息，查阅[这里](../design/coordinator.html)。
Runtime 配置
---------------------

Coordination节点使用的几个全球配置在[配置](../configuration/index.html)和有以下配置设置：
### 节点配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.host`|The host for the current node. This is used to advertise the current processes location as reachable from another node and should generally be specified such that `http://${druid.host}/` could actually talk to this process|InetAddress.getLocalHost().getCanonicalHostName()|
|`druid.port`|This is the port to actually listen on; unless port mapping is used, this will be the same port as is on `druid.host`|8081|
|`druid.service`|The name of the service. This is used as a dimension when emitting metrics and alerts to differentiate between the various services|druid/coordinator|

### Coordinator 操作

|属性|描述|默认|
|--------|-----------|-------|
|`druid.coordinator.period`|The run period for the coordinator. The coordinator’s operates by maintaining the current state of the world in memory and periodically looking at the set of segments available and segments being served to make decisions about whether any changes need to be made to the data topology. This property sets the delay between each of these runs.|PT60S|
|`druid.coordinator.period.indexingPeriod`|How often to send indexing tasks to the indexing service. Only applies if merge or conversion is turned on.|PT1800S (30 mins)|
|`druid.coordinator.startDelay`|The operation of the Coordinator works on the assumption that it has an up-to-date view of the state of the world when it runs, the current ZK interaction code, however, is written in a way that doesn’t allow the Coordinator to know for a fact that it’s done loading the current state of the world. This delay is a hack to give it enough time to believe that it has all the data.|PT300S|
|`druid.coordinator.merge.on`|Boolean flag for whether or not the coordinator should try and merge small segments into a more optimal segment size.|false|
|`druid.coordinator.conversion.on`|Boolean flag for converting old segment indexing versions to the latest segment indexing version.|false|
|`druid.coordinator.load.timeout`|The timeout duration for when the coordinator assigns a segment to a historical node.|PT15M|
|`druid.coordinator.kill.on`|Boolean flag for whether or not the coordinator should submit kill task for unused segments, that is, hard delete them from metadata store and deep storage. If set to true, then for all the whitelisted dataSources, coordinator will submit tasks periodically based on `period` specified. These kill tasks will delete all segments except for the last `durationToRetain` period. Whitelist can be set via dynamic configuration `killDataSourceWhitelist` described later.|false|
|`druid.coordinator.kill.period`|How often to send kill tasks to the indexing service. Value must be greater than `druid.coordinator.period.indexingPeriod`. Only applies if kill is turned on.|PT1D (1 Day)|
|`druid.coordinator.kill.durationToRetain`| Do not kill segments in last `durationToRetain`, must be greater or equal to 0. Only applies and MUST be specified if kill is turned on. Note that default value is invalid.|PT-1S (-1 seconds)|
|`druid.coordinator.kill.maxSegments`|Kill at most n segments per kill task submission, must be greater than 0. Only applies and MUST be specified if kill is turned on. Note that default value is invalid.|0|

### 元数据检索

|属性|描述|默认|
|--------|-----------|-------|
|`druid.manager.config.pollDuration`|How often the manager polls the config table for updates.|PT1m|
|`druid.manager.segments.pollDuration`|The duration between polls the Coordinator does for updates to the set of active segments. Generally defines the amount of lag time it can take for the coordinator to notice new segments.|PT1M|
|`druid.manager.rules.pollDuration`|The duration between polls the Coordinator does for updates to the set of active rules. Generally defines the amount of lag time it can take for the coordinator to notice rules.|PT1M|
|`druid.manager.rules.defaultTier`|The default tier from which default rules will be loaded from.|_default|
|`druid.manager.rules.alertThreshold`|The duration after a failed poll upon which an alert should be emitted.|PT10M|

Dynamic 配置
-----------------------

Coordinator有dynamic配置去改变某些动态行为。Druid[元数据存储](../dependencies/metadata-storage.html)配置表的Coordinator一个JSON规范对象。
建议你使用Coordinator控制台去配置这些参数。然而，如果你需要通过HTTP去配置，JSON对象可以通过POST请求提交到Overlord，在：
```
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config
```

审计配置的可选头参数可以被指定改变。
|头参数名|描述|默认|
|----------|-------------|---------|
|`X-Druid-Author`| author making the config change|""|
|`X-Druid-Comment`| comment describing the change being done|""|

一个Coordinator dynamic配置JSON对象的示例如下：
```json
{
  "millisToWaitBeforeDeleting": 900000,
  "mergeBytesLimit": 100000000L,
  "mergeSegmentsLimit" : 1000,
  "maxSegmentsToMove": 5,
  "replicantLifetime": 15,
  "replicationThrottleLimit": 10,
  "emitBalancingStats": false,
  "killDataSourceWhitelist": ["wikipedia", "testDatasource"]
}
```

在相同的URL发出一个GET请求将返回适当的说明。一个配置设置规格描述如下。

|属性|描述|默认|
|--------|-----------|-------|
|`millisToWaitBeforeDeleting`|How long does the coordinator need to be active before it can start removing (marking unused) segments in metadata storage.|900000 (15 mins)|
|`mergeBytesLimit`|The maximum number of bytes to merge (for segments).|524288000L|
|`mergeSegmentsLimit`|The maximum number of segments that can be in a single [append task](../ingestion/tasks.html).|100|
|`maxSegmentsToMove`|The maximum number of segments that can be moved at any given time.|5|
|`replicantLifetime`|The maximum number of coordinator runs for a segment to be replicated before we start alerting.|15|
|`replicationThrottleLimit`|The maximum number of segments that can be replicated at one time.|10|
|`emitBalancingStats`|Boolean flag for whether or not we should emit balancing stats. This is an expensive operation.|false|
|`killDataSourceWhitelist`|List of dataSources for which kill tasks are sent if property `druid.coordinator.kill.on` is true.|none|

查看Coordinator dynamic配置的审计历史，发出GET请求到URL－
```
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config/history?interval=<interval>
```

间隔默认值可以通过在coordinator runtime.properties设置`druid.audit.manager.auditHistoryMillis`（如果没配置则１周）指定多少。
查看最近n条coordinator dynamic配置的审计员工配置历史，发出GET请求到URL-
```
http://<COORDINATOR_IP>:<PORT>/druid/coordinator/v1/config/history?count=<n>
```
