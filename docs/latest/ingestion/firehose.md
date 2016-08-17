---
layout: doc_page
---

# Druid Firehoses

Firehoses是使用在 [stream-pull](../ingestion/stream-pull.html)的摄取模式。他们是可插入的,因此配置模式和Firehoses的`type` 不同。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | Specifies the type of firehose. Each value will have its own configuration schema, firehoses packaged with Druid are described below. | yes |

## 额外的Firehoses

Druid有几种Firehoses已经是可用的，例如，其他人可以直接在生产环境中使用。

对于额外的Firehoses，请查阅我们的[扩展列表](../development/extensions.html)。
#### LocalFirehose
 
这个Firehoses可以用来从本地磁盘文件读取数据。
这可以为POCs在磁盘上摄取数据。
本地Firehoses规格示例如下：

```json
{
    "type"    : "local",
    "filter"   : "*.csv",
    "baseDir"  : "/data/directory"
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "local".|yes|
|filter|A wildcard filter for files. See [here](http://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/filefilter/WildcardFileFilter.html) for more information.|yes|
|baseDir|directory to search recursively for files to be ingested. |yes|

#### IngestSegmentFirehose

这个Firehoses可以用来从现有的Druid Segment读取数据。
这可以使用一个新的schema和改变Segment的名称，维度，指标，计算等来摄取现有的druid Segment。
摄取Firehoses规格的示例如下-

```json
{
    "type"    : "ingestSegment",
    "dataSource"   : "wikipedia",
    "interval" : "2013-01-01/2013-01-02"
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "ingestSegment".|yes|
|dataSource|A String defining the data source to fetch rows from, very similar to a table in a relational database|yes|
|interval|A String representing ISO-8601 Interval. This defines the time range to fetch the data over.|yes|
|dimensions|The list of dimensions to select. If left empty, no dimensions are returned. If left null or not defined, all dimensions are returned. |no|
|metrics|The list of metrics to select. If left empty, no metrics are returned. If left null or not defined, all metrics are selected.|no|
|filter| See [Filters](../querying/filters.html)|yes|

#### CombiningFirehose

这个Firehoses可以从一列不同的Firehoses结合和合并数据。
这可以从多个Firehoses合并数据。
```json
{
    "type"  :   "combining",
    "delegates" : [ { firehose1 }, { firehose2 }, ..... ]
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "combining"|yes|
|delegates|list of firehoses to combine data from|yes|

#### EventReceiverFirehose

EventReceiverFirehoseFactory可以使用一个http端点摄取事件。
```json
{
  "type": "receiver",
  "serviceName": "eventReceiverServiceName",
  "bufferSize": 10000
}
```

当使用Firehoses时，事件可以通过提交一个POST请求发送到http 端点：
`http://<peonHost>:<port>/druid/worker/v1/chat/<eventReceiverServiceName>/push-events/`

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "receiver"|yes|
|serviceName|name used to announce the event receiver service endpoint|yes|
|bufferSize| size of buffer used by firehose to store events|no default(100000)|

#### TimedShutoffFirehose

这个可以用来启动一个Firehoses，可以在指定时间关闭。
示例如下：

```json
{
    "type"  :   "timed",
    "shutoffTime": "2015-08-25T01:26:05.119Z",
    "delegate": {
          "type": "receiver",
          "serviceName": "eventReceiverServiceName",
          "bufferSize": 100000
     }
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "timed"|yes|
|shutoffTime|time at which the firehose should shut down, in ISO8601 format|yes|
|delegate|firehose to use|yes|
