---
layout: doc_page
---

# 配置Druid

这描述了由所有Druid节点共享的常见配置。这些配置可以在`common.runtime.properties`文件中定义。
## JVM配置最佳实践

这里有四个JVM参数，我们设置启动我们全部的进程：

1.  `-Duser.timezone=UTC` This sets the default timezone of the JVM to UTC. We always set this and do not test with other default timezones, so local timezones might work, but they also might uncover weird and interesting bugs. To issue queries in a non-UTC timezone, see [query granularities](../querying/granularities.html#period-granularities)
2.  `-Dfile.encoding=UTF-8` This is similar to timezone, we test assuming UTF-8. Local encodings might work, but they also might result in weird and interesting bugs.
3.  `-Djava.io.tmpdir=<a path>` Various parts of the system that interact with the file system do it via temporary files, and these files can get somewhat large. Many production systems are set up to have small (but fast) `/tmp` directories, which can be problematic with Druid so we recommend pointing the JVM’s tmp directory to something with a little more meat.
4.  `-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager` This allows log4j2 to handle logs for non-log4j2 components (like jetty) which use standard java logging.

### 扩展

很多Druid的外部依赖可以插入模块。扩展可以使用以下配置：

|属性|描述|默认|
|--------|-----------|-------|
|`druid.extensions.directory`|The root extension directory where user can put extensions related files. Druid will load extensions stored under this directory.|`extensions` (This is a relative path to Druid's working directory)|
|`druid.extensions.hadoopDependenciesDir`|The root hadoop dependencies directory where user can put hadoop related dependencies files. Druid will load the dependencies based on the hadoop coordinate specified in the hadoop index task.|`hadoop-dependencies` (This is a relative path to Druid's working directory|
|`druid.extensions.loadList`|A JSON array of extensions to load from extension directories by Druid. If it is not specified, its value will be `null` and Druid will load all the extensions under `druid.extensions.directory`. If its value is empty list `[]`, then no extensions will be loaded at all.|null|
|`druid.extensions.searchCurrentClassloader`|This is a boolean flag that determines if Druid will search the main classloader for extensions.  It defaults to true but can be turned off if you have reason to not automatically add all modules on the classpath.|true|

### Zookeeper 
我们建议只设置基本ZK路径和ZK服务主机，但Druid使用的ZK路径可以覆盖绝对路径。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.zk.paths.base`|Base Zookeeper path.|`/druid`|
|`druid.zk.service.host`|The ZooKeeper hosts to connect to. This is a REQUIRED property and therefore a host address must be supplied.|none|

#### Zookeeper 行为

|属性|描述|默认|
|--------|-----------|-------|
|`druid.zk.service.sessionTimeoutMs`|ZooKeeper session timeout, in milliseconds.|`30000`|
|`druid.zk.service.compress`|Boolean flag for whether or not created Znodes should be compressed.|`true`|
|`druid.zk.service.acl`|Boolean flag for whether or not to enable ACL security for ZooKeeper. If ACL is enabled, zNode creators will have all permissions.|`false`|

#### 路径配置
Druid通过一组标准路径配置与ZK相互影响。我们建议只设置基本ZK路径，但所有Druid使用的ZK路径可以覆盖绝对路径。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.zk.paths.base`|Base Zookeeper path.|`/druid`|
|`druid.zk.paths.propertiesPath`|Zookeeper properties path.|`${druid.zk.paths.base}/properties`|
|`druid.zk.paths.announcementsPath`|Druid node announcement path.|`${druid.zk.paths.base}/announcements`|
|`druid.zk.paths.liveSegmentsPath`|Current path for where Druid nodes announce their segments.|`${druid.zk.paths.base}/segments`|
|`druid.zk.paths.loadQueuePath`|Entries here cause historical nodes to load and drop segments.|`${druid.zk.paths.base}/loadQueue`|
|`druid.zk.paths.coordinatorPath`|Used by the coordinator for leader election.|`${druid.zk.paths.base}/coordinator`|
|`druid.zk.paths.servedSegmentsPath`|@Deprecated. Legacy path for where Druid nodes announce their segments.|`${druid.zk.paths.base}/servedSegments`|

indexing service也使用它自己的一组路径。这些配置可以包含在常见的配置中。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.zk.paths.indexer.base`|Base zookeeper path for |`${druid.zk.paths.base}/indexer`|
|`druid.zk.paths.indexer.announcementsPath`|Middle managers announce themselves here.|`${druid.zk.paths.indexer.base}/announcements`|
|`druid.zk.paths.indexer.tasksPath`|Used to assign tasks to middle managers.|`${druid.zk.paths.indexer.base}/tasks`|
|`druid.zk.paths.indexer.statusPath`|Parent path for announcement of task statuses.|`${druid.zk.paths.indexer.base}/status`|
|`druid.zk.paths.indexer.leaderLatchPath`|Used for Overlord leader election.|`${druid.zk.paths.indexer.base}/leaderLatchPath`|

如果`druid.zk.paths.base`和`druid.zk.paths.indexer.base`都设置了，而且`druid.zk.paths.*`或者`druid.zk.paths.indexer.*`值都没有设置，那么其他属性将相对于自己的`base`进行设置。例如，如果`druid.zk.paths.base`设置为`/druid1`和`druid.zk.paths.indexer.base`设置为`/druid2`那么`druid.zk.paths.announcementsPath`将默认为`/druid1/announcements`同时`druid.zk.paths.indexer.announcementsPath`默认为`/druid2/announcements`。
以下路径是用于服务搜索。这* *不是* *`druid.zk.paths.base`的影响而且**必须**单独指定。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.discovery.curator.path`|Services announce themselves under this ZooKeeper path.|`/druid/discovery`|

### 启动logging

所有节点可以在启动状态调试日志信息。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.startup.logging.logProperties`|Log all properties on startup (from common.runtime.properties, runtime.properties, and the JVM command line).|false|

注意，如果设置了启动，一些敏感信息可能被日志记录。
### 请求日志

可以查询服务的所有节点也可以log它们看到的查询请求。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.request.logging.type`|Choices: noop, file, emitter. How to log every query request.|noop|

注意，你可以通过设置"io.druid.jetty.RequestLog"到DEBUG级别，启动发送所有的HTTP请求到log。查阅[Logging](../configuration/logging.html)
#### 文件请求logging

日常请求logs存储在disk上。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.request.logging.dir`|Historical, Realtime and Broker nodes maintain request logs of all of the requests they get (interacton is via POST, so normal request logs don’t generally capture information about the actual query), this specifies the directory to store the request logs in|none|

#### Emitter 请求 Logging  

每个请求是emitted到外部的。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.request.logging.feed`|Feed name for requests.|none|

### Enabling Metrics

Druid节点周期性地emit metrics和包括不同的metrics monitors。每个节点可以覆盖默认的monitors列表。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.monitoring.emissionPeriod`|How often metrics are emitted.|PT1m|
|`druid.monitoring.monitors`|Sets list of Druid monitors used by a node. See below for names and more information. For example, you can specify monitors for a Broker with `druid.monitoring.monitors=["com.metamx.metrics.SysMonitor","com.metamx.metrics.JvmMonitor"]`.|none (no monitors)|

下面的monitors是有效的：
|Name|Description|名称|描述|
|----|-----------|
|`io.druid.client.cache.CacheMonitor`|Emits metrics (to logs) about the segment results cache for Historical and Broker nodes. Reports typical cache statistics include hits, misses, rates, and size (bytes and number of entries), as well as timeouts and and errors.|
|`com.metamx.metrics.SysMonitor`|This uses the [SIGAR library](http://www.hyperic.com/products/sigar) to report on various system activities and statuses. Make sure to add the [sigar library jar](https://repository.jboss.org/nexus/content/repositories/thirdparty-uploads/org/hyperic/sigar/1.6.5.132/sigar-1.6.5.132.jar) to your classpath if using this monitor.|
|`io.druid.server.metrics.HistoricalMetricsMonitor`|Reports statistics on Historical nodes.|
|`com.metamx.metrics.JvmMonitor`|Reports JVM-related statistics.|
|`io.druid.segment.realtime.RealtimeMetricsMonitor`|Reports statistics on Realtime nodes.|
|`io.druid.server.metrics.EventReceiverFirehoseMonitor`|Reports how many events have been queued in the EventReceiverFirehose.|

### Emitting Metrics

Druid服务器emit不同的metric和通过一些emitter发出警告。这里有三个emitter包含在代码中，一个“noop” emitter，是只log为log4j（“logging”，是使用默认值如果没有指定的emitter），另一个把JSON事件的POST发送到服务（“http”）。
使用logging emitter的属性描述如下。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.emitter`|Setting this value to "noop", "logging", or "http" will initialize one of the emitter modules. value "composing" can be used to initialize multiple emitter modules. |noop|

#### Logging Emitter Module 

|属性|描述|默认|
|--------|-----------|-------|
|`druid.emitter.logging.loggerClass`|Choices: HttpPostEmitter, LoggingEmitter, NoopServiceEmitter, ServiceEmitter. The class used for logging.|LoggingEmitter|
|`druid.emitter.logging.logLevel`|Choices: debug, info, warn, error. The log level at which message are logged.|info|

#### Http Emitter Module

|属性|描述|默认|
|--------|-----------|-------|
|`druid.emitter.http.timeOut`|The timeout for data reads.|PT5M|
|`druid.emitter.http.flushMillis`|How often the internal message buffer is flushed (data is sent).|60000|
|`druid.emitter.http.flushCount`|How many messages the internal message buffer can hold before flushing (sending).|500|
|`druid.emitter.http.recipientBaseUrl`|The base URL to emit messages to. Druid will POST JSON to be consumed at the HTTP endpoint specified by this property.|none|

#### 组合Emitter模块

|属性|描述|默认|
|--------|-----------|-------|
|`druid.emitter.composing.emitters`|List of emitter modules to load e.g. ["logging","http"].|[]|

#### Graphite Emitter

使用graphite作为Emitter，设置`druid.emitter=graphite`。  配置细节请看下面这个[链接](https://github.com/druid-io/druid/tree/master/extensions/graphite-emitter/README.md)。

### Metadata 存储

这些属性指定jdbc链接和metadata存储的其他配置。唯一把metadata存储和这些属性连接的进程是[Coordinator](../design/coordinator.html)，[Indexing service](../design/indexing-service.html)和[Realtime节点](../design/realtime.html)。


|属性|描述|默认|
|--------|-----------|-------|
|`druid.metadata.storage.type`|The type of metadata storage to use. Choose from "mysql", "postgresql", or "derby".|derby|
|`druid.metadata.storage.connector.connectURI`|The jdbc uri for the database to connect to|none|
|`druid.metadata.storage.connector.user`|The username to connect with.|none|
|`druid.metadata.storage.connector.password`|The password to connect with.|none|
|`druid.metadata.storage.connector.createTables`|If Druid requires a table and it doesn't exist, create it?|true|
|`druid.metadata.storage.tables.base`|The base name for tables.|druid|
|`druid.metadata.storage.tables.segments`|The table to use to look for segments.|druid_segments|
|`druid.metadata.storage.tables.rules`|The table to use to look for segment load/drop rules.|druid_rules|
|`druid.metadata.storage.tables.config`|The table to use to look for configs.|druid_config|
|`druid.metadata.storage.tables.tasks`|Used by the indexing service to store tasks.|druid_tasks|
|`druid.metadata.storage.tables.taskLog`|Used by the indexing service to store task logs.|druid_taskLog|
|`druid.metadata.storage.tables.taskLock`|Used by the indexing service to store task locks.|druid_taskLock|
|`druid.metadata.storage.tables.audit`|The table to use for audit history of configuration changes e.g. Coordinator rules.|druid_audit|

### Deep Storage 存储

关注如何从deep存储推和拉[Segments](../design/segments.html)的配置。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.storage.type`|Choices:local, noop, s3, hdfs, c*. The type of deep storage to use.|local|

#### 本地deep存储

本地deep存储使用本地文件系统。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.storage.storageDirectory`|Directory on disk to use as deep storage.|/tmp/druid/localStorage|

#### Noop Deep 存储

这个deep存储不需要做任何事。它没有配置。
#### S3 Deep Storage

这个deep存储是使用Amazon's S3 接口。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.s3.accessKey`|The access key to use to access S3.|none|
|`druid.s3.secretKey`|The secret key to use to access S3.|none|
|`druid.storage.bucket`|S3 bucket name.|none|
|`druid.storage.baseKey`|S3 object key prefix for storage.|none|
|`druid.storage.disableAcl`|Boolean flag for ACL.|false|
|`druid.storage.archiveBucket`|S3 bucket name for archiving when running the indexing-service *archive task*.|none|
|`druid.storage.archiveBaseKey`|S3 object key prefix for archiving.|none|

#### HDFS Deep Storage

这个deep存储是使用HDFS接口。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.storage.storageDirectory`|HDFS directory to use as deep storage.|none|

#### Cassandra Deep Storage

这个deep存储是使用Cassandra接口。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.storage.host`|Cassandra host.|none|
|`druid.storage.keyspace`|Cassandra key space.|none|

### Caching缓存

你可以使用下面的配置启动在Broker，Historical，或者realtime级别的结果缓存。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.type`|`local`, `memcached`|The type of cache to use for queries.|`local`|
|`druid.(broker|historical|realtime).cache.unCacheable`|All druid query types|All query types to not cache.|["groupBy", "select"]|
|`druid.(broker|historical|realtime).cache.useCache`|Whether to use cache for getting query results.|false|
|`druid.(broker|historical|realtime).cache.populateCache`|Whether to populate cache.|false|

#### 本地缓存

|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.sizeInBytes`|Maximum cache size in bytes. You must set this if you enabled populateCache/useCache, or else cache size of zero wouldn't really cache anything.|0|
|`druid.cache.initialSize`|Initial size of the hashtable backing the cache.|500000|
|`druid.cache.logEvictionCount`|If non-zero, log cache eviction every `logEvictionCount` items.|0|

#### Memcache

|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.expiration`|Memcached [expiration time](https://code.google.com/p/memcached/wiki/NewCommands#Standard_Protocol).|2592000 (30 days)|
|`druid.cache.timeout`|Maximum time in milliseconds to wait for a response from Memcached.|500|
|`druid.cache.hosts`|Command separated list of Memcached hosts `<host:port>`.|none|
|`druid.cache.maxObjectSize`|Maximum object size in bytes for a Memcached object.|52428800 (50 MB)|
|`druid.cache.memcachedPrefix`|Key prefix for all keys in Memcached.|druid|

### Indexing Service Discovery

这个配置是使用来查找[Indexing Service](../design/indexing-service.html) 用Curator service discovery。只需要你实际运行一个indexing service。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.selectors.indexing.serviceName`|The druid.service name of the indexing service Overlord node. To start the Overlord with a different name, set it with this property. |druid/overlord|


### Coordinator Discovery

这个配置是用Curator service discovery来查找[Coordinator](../design/coordinator.html)。这个配置是通过Realtime indexing 节点去获得集群上segment加载的信息。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.selectors.coordinator.serviceName`|The druid.service name of the coordinator node. To start the Coordinator with a different name, set it with this property. |druid/coordinator|


### 发布Segment

你可以配置怎么发布和在Zookeeper（使用curator）上不发布Znodes。对于正常操作你不需要去覆盖这些配置。
##### 批量数据Segment announcer

当前的Druid，多个数据Segment可能在相同的Znode下announced。 
|属性|描述|默认|
|--------|-----------|-------|
|`druid.announcer.segmentsPerNode`|Each Znode contains info for up to this many segments.|50|
|`druid.announcer.maxBytesPerNode`|Max byte size for Znode.|524288|
