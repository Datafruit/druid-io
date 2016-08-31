---
layout: doc_page
---
# Druid 指标

Druid生成与查询，摄取，协调相关的度量。
度量在运行日志文件或通过HTTP(Apache Kafka等服务）时作为JSON对象发出。度量默认情况下是禁用的。
所有的Druid指标公用一个常见的字段组：
* `时间戳` - 创建指标的时间
* `指标` - 指标名称
* `服务` - 发出指标的服务名称
* `主机` - 发出指标的主机名
* `值` - 与指标相关的一些数值

指标可能有上面这些列表以外的维度
大多数指标值重置每个发行时间。默认情况下Druid发行时间是1分钟，这个可以通过设置属性`druid.monitoring.emissionPeriod`来改变。
有效的指标
-----------------

## 查询指标

### 代理

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s|
|`query/bytes`|number of bytes returned in query response.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.| |
|`query/node/time`|Milliseconds taken to query individual historical/realtime nodes.|id, status, server.|< 1s|
|`query/node/bytes`|number of bytes returned from querying individual historical/realtime nodes.|id, status, server.| |
|`query/node/ttfb`|Time to first byte. Milliseconds elapsed until broker starts receiving the response from individual historical/realtime nodes.|id, status, server.|< 1s|
|`query/intervalChunk/time`|Only emitted if interval chunking is enabled. Milliseconds required to query an interval chunk.|id, status, chunkInterval (if interval chunking is enabled).|< 1s|

### 历史的

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s|
|`query/segment/time`|Milliseconds taken to query individual segment. Includes time to page in the segment from disk.|id, status, segment.|several hundred milliseconds|
|`query/wait/time`|Milliseconds spent waiting for a segment to be scanned.|id, segment.|< several hundred milliseconds|
|`segment/scan/pending`|Number of segments in queue waiting to be scanned.||Close to 0|
|`query/segmentAndCache/time`|Milliseconds taken to query individual segment or hit the cache (if it is enabled on the historical node).|id, segment.|several hundred milliseconds|
|`query/cpu/time`|Microseconds of CPU time taken to complete a query|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|Varies|

### 实时

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s|
|`query/wait/time`|Milliseconds spent waiting for a segment to be scanned.|id, segment.|several hundred milliseconds|
|`segment/scan/pending`|Number of segments in queue waiting to be scanned.||Close to 0|

### 缓存

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`query/cache/delta/*`|Cache metrics since the last emission.||N/A|
|`query/cache/total/*`|Total cache metrics.||N/A|


|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`*/numEntries`|Number of cache entries.||Varies.|
|`*/sizeBytes`|Size in bytes of cache entries.||Varies.|
|`*/hits`|Number of cache hits.||Varies.|
|`*/misses`|Number of cache misses.||Varies.|
|`*/evictions`|Number of cache evictions.||Varies.|
|`*/hitRate`|Cache hit rate.||~40%|
|`*/averageByte`|Average cache entry byte size.||Varies.|
|`*/timeouts`|Number of cache timeouts.||0|
|`*/errors`|Number of cache errors.||0|

#### 分布式缓存唯一指标

分布式缓存客户端指标按以下记录。这些指标直接来自客户端而不是缓存中检索层。
|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`query/cache/memcached/total`|Cache metrics unique to memcached (only if `druid.cache.type=memcached`) as their actual values|Variable|N/A|
|`query/cache/memcached/delta`|Cache metrics unique to memcached (only if `druid.cache.type=memcached`) as their delta from the prior event emission|Variable|N/A|


## 摄取指标

这些指标只有实时指标监听包含在监听列表中时有效。这些指标为每个发行期间delta。

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`ingest/events/thrownAway`|Number of events rejected because they are outside the windowPeriod.|dataSource.|0|
|`ingest/events/unparseable`|Number of events rejected because the events are unparseable.|dataSource.|0|
|`ingest/events/processed`|Number of events successfully processed per emission period.|dataSource.|Equal to your # of events per emission period.|
|`ingest/rows/output`|Number of Druid rows persisted.|dataSource.|Your # of events with rollup.|
|`ingest/persists/count`|Number of times persist occurred.|dataSource.|Depends on configuration.|
|`ingest/persists/time`|Milliseconds spent doing intermediate persist.|dataSource.|Depends on configuration. Generally a few minutes at most.|
|`ingest/persists/cpu`|Cpu time in Nanoseconds spent on doing intermediate persist.|dataSource.|Depends on configuration. Generally a few minutes at most.|
|`ingest/persists/backPressure`|Number of persists pending.|dataSource.|0|
|`ingest/persists/failed`|Number of persists that failed.|dataSource.|0|
|`ingest/handoff/failed`|Number of handoffs that failed.|dataSource.|0|
|`ingest/merge/time`|Milliseconds spent merging intermediate segments|dataSource.|Depends on configuration. Generally a few minutes at most.|
|`ingest/merge/cpu`|Cpu time in Nanoseconds spent on merging intermediate segments.|dataSource.|Depends on configuration. Generally a few minutes at most.|
|`ingest/handoff/count`|Number of handoffs that happened.|dataSource.|Varies. Generally greater than 0 once every segment granular period if cluster operating normally|
 
注意：如果JVM不支持当前线程的CPU时间测量，摄取/合并/cpu和摄取/坚持/cpu 将为 0。
### 索引服务

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`task/run/time`|Milliseconds taken to run task.|dataSource, taskType, taskStatus.|Varies.|
|`segment/added/bytes`|Size in bytes of new segments created.|dataSource, taskType, interval.|Varies.|
|`segment/moved/bytes`|Size in bytes of segments moved/archived via the Move Task.|dataSource, taskType, interval.|Varies.|
|`segment/nuked/bytes`|Size in bytes of segments deleted via the Kill Task.|dataSource, taskType, interval.|Varies.|

## 协调

这些指标是为了Druid协调器和能重置每次运行协调逻辑的协调器。

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`segment/assigned/count`|Number of segments assigned to be loaded in the cluster.|tier.|Varies.|
|`segment/moved/count`|Number of segments moved in the cluster.|tier.|Varies.|
|`segment/dropped/count`|Number of segments dropped due to being overshadowed.|tier.|Varies.|
|`segment/deleted/count`|Number of segments dropped due to rules.|tier.|Varies.|
|`segment/unneeded/count`|Number of segments dropped due to being marked as unused.|tier.|Varies.|
|`segment/cost/raw`|Used in cost balancing. The raw cost of hosting segments.|tier.|Varies.|
|`segment/cost/normalization`|Used in cost balancing. The normalization of hosting segments.|tier.|Varies.|
|`segment/cost/normalized`|Used in cost balancing. The normalized cost of hosting segments.|tier.|Varies.|
|`segment/loadQueue/size`|Size in bytes of segments to load.|server.|Varies.|
|`segment/loadQueue/failed`|Number of segments that failed to load.|server.|0|
|`segment/loadQueue/count`|Number of segments to load.|server.|Varies.|
|`segment/dropQueue/count`|Number of segments to drop.|server.|Varies.|
|`segment/size`|Size in bytes of available segments.|dataSource.|Varies.|
|`segment/count`|Number of available segments.|dataSource.|< max|
|`segment/overShadowed/count`|Number of overShadowed segments.||Varies.|

## 一般正常

### 历史的

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`segment/max`|Maximum byte limit available for segments.||Varies.|
|`segment/used`|Bytes used for served segments.|dataSource, tier, priority.|< max|
|`segment/usedPercent`|Percentage of space used by served segments.|dataSource, tier, priority.|< 100%|
|`segment/count`|Number of served segments.|dataSource, tier, priority.|Varies.|

### JVM

这些指标只有当JVMMonitor 模块包含在内时才有效。

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`jvm/pool/committed`|Committed pool.|poolKind, poolName.|close to max pool|
|`jvm/pool/init`|Initial pool.|poolKind, poolName.|Varies.|
|`jvm/pool/max`|Max pool.|poolKind, poolName.|Varies.|
|`jvm/pool/used`|Pool used.|poolKind, poolName.|< max pool|
|`jvm/bufferpool/count`|Bufferpool count.|bufferPoolName.|Varies.|
|`jvm/bufferpool/used`|Bufferpool used.|bufferPoolName.|close to capacity|
|`jvm/bufferpool/capacity`|Bufferpool capacity.|bufferPoolName.|Varies.|
|`jvm/mem/init`|Initial memory.|memKind.|Varies.|
|`jvm/mem/max`|Max memory.|memKind.|Varies.|
|`jvm/mem/used`|Used memory.|memKind.|< max memory|
|`jvm/mem/committed`|Committed memory.|memKind.|close to max memory|
|`jvm/gc/count`|Garbage collection count.|gcName.|< 100|
|`jvm/gc/time`|Garbage collection time.|gcName.|< 1s|	

### EventReceiverFirehose

下面的指标只有当EventReceiverFirehoseMonitor 模块包含在内时才有效。

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`ingest/events/buffered`|Number of events queued in the EventReceiverFirehose's buffer|serviceName, dataSource, taskId, bufferCapacity.|Equal to current # of events in the buffer queue.|
|`ingest/bytes/received`|Number of bytes received by the EventReceiverFirehose.|serviceName, dataSource, taskId.|Varies.|

## Sys

这些指标只有当SysMonitor模块包括在内时才有效。

|指标|描述|维度|正常值|
|------|-----------|----------|------------|
|`sys/swap/free`|Free swap.||Varies.|
|`sys/swap/max`|Max swap.||Varies.|
|`sys/swap/pageIn`|Paged in swap.||Varies.|
|`sys/swap/pageOut`|Paged out swap.||Varies.|
|`sys/disk/write/count`|Writes to disk.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.|
|`sys/disk/read/count`|Reads from disk.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.|
|`sys/disk/write/size`|Bytes written to disk. Can we used to determine how much paging is occuring with regards to segments.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.|
|`sys/disk/read/size`|Bytes read from disk. Can we used to determine how much paging is occuring with regards to segments.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.|
|`sys/net/write/size`|Bytes written to the network.|netName, netAddress, netHwaddr|Varies.|
|`sys/net/read/size`|Bytes read from the network.|netName, netAddress, netHwaddr|Varies.|
|`sys/fs/used`|Filesystem bytes used.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|< max|
|`sys/fs/max`|Filesystesm bytes max.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.|
|`sys/mem/used`|Memory used.||< max|
|`sys/mem/max`|Memory max.||Varies.|
|`sys/storage/used`|Disk space used.|fsDirName.|Varies.|
|`sys/cpu`|CPU used.|cpuName, cpuTime.|Varies.|
