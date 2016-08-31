---
layout: doc_page
---
# 更新现有的数据

对于时间间隔并创建Druid段，一旦摄入一些数据源中的数据,您可能想要更改摄入数据。有几种方法可以做到这一点。

##### 更新维度值

如果你有一个维度值需要频繁更新,首先试试使用[查找](../querying/lookups.html)。
一个查找的典型用例是当你有一个ID维度存储在一个Druid段,并希望ID维度映射到一个人们可读的字符串值,可能需要定期更新。
##### 重建段（重建索引）

如果查询充分,你可以完全重建Druid段特定的间隔时间。
重建一个段被称为重建索引数据。举个例子,如果你想添加或删除列从你现有的段,或者你想改变你的汇总粒度段,您将不得不重建索引数据。
我们建议保持你的原始数据的一个副本,以防你需要重建索引数据。
##### 处理延迟事件(Delta摄入)

如果你有一批摄入管道和延迟事件进来,想将这些事件附加到现有的段,为避免重建新的段重建索引的开销,您可以使用delta摄入。
### 使用Hadoop批摄入重建索引和delta摄入

本节假设读者知道如何做批处理使用Hadoop摄入。
查阅[批摄取](batch-ingestion.html)了解更多信息。Hadoop 批摄取可用于重建索引和delta摄入。

Druid在`ioConfig`中使用一个`inputSpec`了解摄取的数据所在以及如何读它。
对于简单的Hadoop批摄入,`静态`或`粒度`规范类型允许您在深存储读取数据存储。
还有其他类型的`inputSpec`使用重建索引和delta摄入。
#### `数据源`

这是一种`inputSpec`,读取数据已经存储在Druid。

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|type|String.|This should always be 'dataSource'.|yes|
|ingestionSpec|JSON object.|Specification of Druid segments to be loaded. See below.|yes|
|maxSplitSize|Number|Enables combining multiple segments into single Hadoop InputSplit according to size of segments. Default is none. |no|
 
`ingestionSpec`的内容如下：  

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|dataSource|String|Druid dataSource name from which you are loading the data.|yes|
|intervals|List|A list of strings representing ISO-8601 Intervals.|yes|
|granularity|String|Defines the granularity of the query while loading data. Default value is "none". See [Granularities](../querying/granularities.html).|no|
|filter|JSON|See [Filters](../querying/filters.html)|no|
|dimensions|Array of String|Name of dimension columns to load. By default, the list will be constructed from parseSpec. If parseSpec does not have an explicit list of dimensions then all the dimension columns present in stored data will be read.|no|
|metrics|Array of String|Name of metric columns to load. By default, the list will be constructed from the "name" of all the configured aggregators.|no|
|ignoreWhenNoSegments|boolean|Whether to ignore this ingestionSpec if no segments were found. Default behavior is to throw error when no segments were found.|no|

示例

```json
"ioConfig" : {
  "type" : "hadoop",
  "inputSpec" : {
    "type" : "dataSource",
    "ingestionSpec" : {
      "dataSource": "wikipedia",
      "intervals": ["2014-10-20T00:00:00Z/P2W"]
    }
  },
  ...
}
```

#### `multi`

这是一个组合inputSpec与其他inputSpecs结合。这个inputSpec用于delta摄入。
请注意,delta摄入并不是一个幂等操作。我们将来可以增加一些改变使得它幂等。
|字段|类型|描述|必须|
|-----|----|-----------|--------|
|children|Array of JSON objects|List of JSON objects containing other inputSpecs.|yes|

示例：
```json
"ioConfig" : {
  "type" : "hadoop",
  "inputSpec" : {
    "type" : "multi",
    "children": [
      {
        "type" : "dataSource",
        "ingestionSpec" : {
          "dataSource": "wikipedia",
          "intervals": ["2014-10-20T00:00:00Z/P2W"]
        }
      },
      {
        "type" : "static",
        "paths": "/path/to/more/wikipedia/data/"
      }
    ]  
  },
  ...
}
```

### 重建索引没有Hadoop批摄入

这部分假设读者理解使用[IndexTask](../ingestion/tasks.html#index-task)不用Hadoop怎样批摄取，这个是用了一个“Firehoses“来了解输入的数据所在和怎么读取。
[IngestSegmentFirehose](firehose.html#ingestsegmentfirehose)可以从Druid里的段读取数据。 
注意,索引任务用于原型的目的只有这所有的处理在一个过程。生产场景处理超过1GB的数据请使用Hadoop批量摄入。
