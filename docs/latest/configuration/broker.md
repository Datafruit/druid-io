---
layout: doc_page
---
broker节点配置
=========================
一般的broker节点信息，查阅[这里](../design/broker.html)。
Runtime Configuration
---------------------

broker节点使用的几个全球配置在[配置](../configuration/index.html)和有以下配置设置：
### 节点配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.host`|The host for the current node. This is used to advertise the current processes location as reachable from another node and should generally be specified such that `http://${druid.host}/` could actually talk to this process|InetAddress.getLocalHost().getCanonicalHostName()|
|`druid.port`|This is the port to actually listen on; unless port mapping is used, this will be the same port as is on `druid.host`|8082|
|`druid.service`|The name of the service. This is used as a dimension when emitting metrics and alerts to differentiate between the various services|druid/broker|

### 查询配置

#### 查询优化

|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.broker.balancer.type`|`random`, `connectionCount`|Determines how the broker balances connections to historical nodes. `random` choose randomly, `connectionCount` picks the node with the fewest number of active connections to|`random`|
|`druid.broker.select.tier`|`highestPriority`, `lowestPriority`, `custom`|If segments are cross-replicated across tiers in a cluster, you can tell the broker to prefer to select segments in a tier with a certain priority.|`highestPriority`|
|`druid.broker.select.tier.custom.priorities`|`An array of integer priorities.`|Select servers in tiers with a custom priority list.|None|

#### 并发请求

Druid用Jetty为HTTP请求服务。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.server.http.numThreads`|Number of threads for HTTP requests.|10|
|`druid.server.http.maxIdleTime`|The Jetty max idle time for a connection.|PT5m|
|`druid.broker.http.numConnections`|Size of connection pool for the Broker to connect to historical and real-time nodes. If there are more queries than this number that all need to speak to the same node, then they will queue up.|5|
|`druid.broker.http.readTimeout`|The timeout for data reads.|PT15M|

#### Retry Policy重试策略

Druid Broker可以为短暂的错误选择重试内部查询。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.broker.retryPolicy.numTries`|Number of tries.|1|

#### 处理

broker为嵌套groupBy查询使用处理配置。可选地，长间隔查询(任何类型的)可以分为短区间查询和并行处理这个线程池。更多细节,请参阅“chunkPeriod”在[查询文档](../querying/query-context.html)文档。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.processing.buffer.sizeBytes`|This specifies a buffer size for the storage of intermediate results. The computation engine in both the Historical and Realtime nodes will use a scratch buffer of this size to do all of their intermediate computations off-heap. Larger values allow for more aggregations in a single pass over the data while smaller values can require more passes depending on the query that is being executed.|1073741824 (1GB)|
|`druid.processing.buffer.poolCacheMaxCount`|processing buffer pool caches the buffers for later use, this is the maximum count cache will grow to. note that pool can create more buffers than it can cache if necessary.|Integer.MAX_VALUE|
|`druid.processing.formatString`|Realtime and historical nodes use this format string to name their processing threads.|processing-%s|
|`druid.processing.numThreads`|The number of processing threads to have available for parallel processing of segments. Our rule of thumb is `num_cores - 1`, which means that even under heavy load there will still be one core available to do background tasks like talking with ZooKeeper and pulling down segments. If only one core is available, this property defaults to the value `1`.|Number of cores - 1 (or 1)|
|`druid.processing.columnCache.sizeBytes`|Maximum size in bytes for the dimension value lookup cache. Any value greater than `0` enables the cache. It is currently disabled by default. Enabling the lookup cache can significantly improve the performance of aggregators operating on dimension values, such as the JavaScript aggregator, or cardinality aggregator, but can slow things down if the cache hit rate is low (i.e. dimensions with few repeating values). Enabling it may also require additional garbage collection tuning to avoid long GC pauses.|`0` (disabled)|
|`druid.processing.fifo`|If the processing queue should treat tasks of equal priority in a FIFO manner|`false`|

#### 通用查询配置

##### GroupBy查询配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.query.groupBy.singleThreaded`|Run single threaded group By queries.|false|
|`druid.query.groupBy.maxIntermediateRows`|Maximum number of intermediate rows.|50000|
|`druid.query.groupBy.maxResults`|Maximum number of results.|500000|

##### 搜索查询配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.query.search.maxSearchLimit`|Maximum number of search results to return.|1000|

##### 段元数据查询配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.query.segmentMetadata.defaultHistory`|When no interval is specified in the query, use a default interval of defaultHistory before the end time of the most recent segment, specified in ISO8601 format. This property also controls the duration of the default interval used by GET /druid/v2/datasources/{dataSourceName} interactions for retrieving datasource dimensions/metrics.|P1W|

### 缓存

在这里你可以只选择配置缓存，通过设置缓存配置启用代理
|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.broker.cache.useCache`|true, false|Enable the cache on the broker.|false|
|`druid.broker.cache.populateCache`|true, false|Populate the cache on the broker.|false|
|`druid.broker.cache.unCacheable`|All druid query types|All query types to not cache.|`["groupBy", "select"]`|
|`druid.broker.cache.cacheBulkMergeLimit`|positive integer or 0|Queries with more segments than this number will not attempt to fetch from cache at the broker level, leaving potential caching fetches (and cache result merging) to the historicals|`Integer.MAX_VALUE`|

查阅[缓存配置](caching.html)了解缓存设置怎样配置。