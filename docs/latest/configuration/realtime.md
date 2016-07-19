---
layout: doc_page
---

Realtime Node 配置
================
Realtime Node的一般信息，查阅[这里](../design/realtime.html)。
Runtime 配置
---------------------

Realtime node 使用的几个全球配置在 [配置](../configuration/index.html)和具有以下配置： 
### Node 配置

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.host`|The host for the current node. This is used to advertise the current processes location as reachable from another node and should generally be specified such that `http://${druid.host}/` could actually talk to this process|InetAddress.getLocalHost().getCanonicalHostName()|
|`druid.port`|This is the port to actually listen on; unless port mapping is used, this will be the same port as is on `druid.host`|8084|
|`druid.service`|The name of the service. This is used as a dimension when emitting metrics and alerts to differentiate between the various services|druid/realtime|

### 操作

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.publish.type`|Where to publish segments. Choices are "noop" or "metadata".|metadata|
|`druid.realtime.specFile`|File location of realtime specFile.|none|

### 存储中间段

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.segmentCache.locations`|Where intermediate segments are stored. The maxSize should always be zero.|none|


### 查询配置

#### 处理

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.processing.buffer.sizeBytes`|This specifies a buffer size for the storage of intermediate results. The computation engine in both the Historical and Realtime nodes will use a scratch buffer of this size to do all of their intermediate computations off-heap. Larger values allow for more aggregations in a single pass over the data while smaller values can require more passes depending on the query that is being executed.|1073741824 (1GB)|
|`druid.processing.formatString`|Realtime and historical nodes use this format string to name their processing threads.|processing-%s|
|`druid.processing.numThreads`|The number of processing threads to have available for parallel processing of segments. Our rule of thumb is `num_cores - 1`, which means that even under heavy load there will still be one core available to do background tasks like talking with ZooKeeper and pulling down segments. If only one core is available, this property defaults to the value `1`.|Number of cores - 1 (or 1)|
|`druid.processing.columnCache.sizeBytes`|Maximum size in bytes for the dimension value lookup cache. Any value greater than `0` enables the cache. It is currently disabled by default. Enabling the lookup cache can significantly improve the performance of aggregators operating on dimension values, such as the JavaScript aggregator, or cardinality aggregator, but can slow things down if the cache hit rate is low (i.e. dimensions with few repeating values). Enabling it may also require additional garbage collection tuning to avoid long GC pauses.|`0` (disabled)|

#### 一般查询配置

##### GroupBy查询配置

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.query.groupBy.singleThreaded`|Run single threaded group By queries.|false|
|`druid.query.groupBy.maxIntermediateRows`|Maximum number of intermediate rows.|50000|
|`druid.query.groupBy.maxResults`|Maximum number of results.|500000|

##### Search Query Config 搜索查询配置

|属性|描述|默认值|
|--------|-----------|-------|
|`druid.query.search.maxSearchLimit`|Maximum number of search results to return.|1000|

### 缓存

你可以通过设置缓存配置在实时节点上选择启用配置缓存
|属性|可能值|描述|默认值|
|--------|---------------|-----------|-------|
|`druid.realtime.cache.useCache`|true, false|Enable the cache on the realtime.|false|
|`druid.realtime.cache.populateCache`|true, false|Populate the cache on the realtime.|false|
|`druid.realtime.cache.unCacheable`|All druid query types|All query types to not cache.|`["groupBy", "select"]`|

查看[缓存](caching.html) 了解怎么配置缓存设置。