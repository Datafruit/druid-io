---
layout: doc_page
---
# 任务
任务运行在中层管理者而且总是运行在单一数据源中。任务提交使用[POST请求](../design/indexing-service.html)。

这里有几种不同的任务类型。
段创建任务
----------------------

### 索引任务

查阅[批摄取](../ingestion/batch-ingestion.html)。
### 索引任务

索引任务是索引Hadoop任务更简单的变异,被设计用于较小的数据集。索引服务内执行任务,不需要外部Hadoop设置。索引任务的语法如下:
```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "column" : "timestamp",
            "format" : "auto"
          },
          "dimensionsSpec" : {
            "dimensions": ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"],
            "dimensionExclusions" : [],
            "spatialDimensions" : []
          }
        }
      },
      "metricsSpec" : [
        {
          "type" : "count",
          "name" : "count"
        },
        {
          "type" : "doubleSum",
          "name" : "added",
          "fieldName" : "added"
        },
        {
          "type" : "doubleSum",
          "name" : "deleted",
          "fieldName" : "deleted"
        },
        {
          "type" : "doubleSum",
          "name" : "delta",
          "fieldName" : "delta"
        }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "DAY",
        "queryGranularity" : "NONE",
        "intervals" : [ "2013-08-31/2013-09-01" ]
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "examples/indexing/",
        "filter" : "wikipedia_data.json"
       }
    },
    "tuningConfig" : {
      "type" : "index",
      "targetPartitionSize" : -1,
      "rowFlushBoundary" : 0,
      "numShards": 1
    }
  }
}
```

#### 任务属性

|属性|描述|必须?|
|--------|-----------|---------|
|type|The task type, this should always be "index".|yes|
|id|The task ID. If this is not explicitly specified, Druid generates the task ID using the name of the task file and date-time stamp. |no|
|spec|The ingestion spec. See below for more details. |yes|

#### DataSchema

这个字段是要求的。
查阅[摄取](../ingestion/index.html)
#### IOConfig

这个字段是必须的，你可以在这里指定一种[Firehose](../ingestion/firehose.html)类型。
#### 优化配置

优化配置是可选的,如果没有指定tuningConfig将使用缺省参数。见下文的更多细节。
|属性|描述|默认|必须?|
|--------|-----------|-------|---------|
|type|The task type, this should always be "index".|None.|yes|
|targetPartitionSize|Used in sharding. Determines how many rows are in each segment. Set this to -1 to use numShards instead for sharding.|5000000|no|
|rowFlushBoundary|Used in determining when intermediate persist should occur to disk.|500000|no|
|numShards|Directly specify the number of shards to create. You can skip the intermediate persist step if you specify the number of shards you want and set targetPartitionSize=-1.|null|no|
|indexSpec|defines segment storage format options to be used at indexing time, see [IndexSpec](#indexspec)|null|no|

#### 索引规范

索引规范在索引时定义了段存储格式选项，如位图类型和列压缩格式。

索引规范是可选的,如果没有指定将使用缺省参数。
|属性|描述|可能的值|默认值|必须?|
|--------|-----------|---------------|-------|---------|
|bitmap|type of bitmap compression to use for inverted indices.|`"concise"`, `"roaring"`|`"concise"`|no|
|dimensionCompression|compression format for dimension columns|`"uncompressed"`, `"lz4"`, `"lzf"`|`"lz4"`|no|
|metricCompression|compression format for metric columns, defaults to LZ4|`"lz4"`, `"lzf"`|`"lz4"`|no|

段合并任务
--------------------------

### 追加任务
 
追加任务和一列段到一个单段（一个接着一个）。语法如下：
```json
{
    "type": "append",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "segments": <JSON list of DataSegment objects to append>,
    "aggregations": <optional list of aggregators>
}
```

### 合并任务
 
合并任务和一列段。任何常见的时间戳都是可以合并的。语法如下：
```json
{
    "type": "merge",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "aggregations": <list of aggregators>,
    "segments": <JSON list of DataSegment objects to merge>
}
```

 段破坏任务
------

### 强制关闭任务

强制关闭任务从深存储删除段的所有信息和删除该任务。在Druid 段表中必须禁用Killable段(used==0)。有效的语法是:
```json
{
    "type": "kill",
    "id": <task_id>,
    "dataSource": <task_datasource>,
    "interval" : <all_segments_in_this_interval_will_die!>
}
```
 
混合任务
----

### 版本转换任务
任务转换套件需要活动段和将使用新的索引规范从而不压缩他们。这是非常方便的操作,比如从简洁迁移到复杂时,或添加维度压缩旧段。
新段将有相同版本的旧段`_converted`附加。转换任务对于多次相同的数据源可能运行在不同的时间间隔。
每个执行将添加另一个“_converted”部分的版本。
有两种类型转换任务。一个是Hadoop转换任务,另一个是索引服务转换任务。Hadoop转换任务运行在一个Hadoop集群,只是留下一个任务监控到索引服务(类似于hadoop批处理任务)。索引服务转换任务是在索引服务运行实际的转换。

#### 转换段任务
```json
{
  "type": "hadoop_convert_segment",
  "dataSource":"some_datasource",
  "interval":"2013/2015",
  "indexSpec":{"bitmap":{"type":"concise"},"dimensionCompression":"lz4","metricCompression":"lz4"},
  "force": true,
  "validate": false,
  "distributedSuccessCache":"hdfs://some-hdfs-nn:9000/user/jobrunner/cache",
  "jobPriority":"VERY_LOW",
  "segmentOutputPath":"s3n://somebucket/somekeyprefix"
}
```

这些值描述如下。

|属性|类型|描述|必须?|
|-----|----|-----------|--------|
|`type`|String|Convert task identifier|Yes: `hadoop_convert_segment`|
|`dataSource`|String|The datasource to search for segments|Yes|
|`interval`|Interval string|The interval in the datasource to look for segments|Yes|
|`indexSpec`|json|The compression specification for the index|Yes|
|`force`|boolean|Forces the convert task to continue even if binary versions indicate it has been updated recently (you probably want to do this)|No|
|`validate`|boolean|Runs validation between the old and new segment before reporting task success|No|
|`distributedSuccessCache`|URI|A location where hadoop should put intermediary files.|Yes|
|`jobPriority`|`org.apache.hadoop.mapred.JobPriority` as String|The priority to set for the hadoop job|No|
|`segmentOutputPath`|URI|A base uri for the segment to be placed. Same format as other places a segment output path is needed|Yes|


#### 索引服务转换段任务
```json
{
  "type": "convert_segment",
  "dataSource":"some_datasource",
  "interval":"2013/2015",
  "indexSpec":{"bitmap":{"type":"concise"},"dimensionCompression":"lz4","metricCompression":"lz4"},
  "force": true,
  "validate": false
}
```
|字段|类型|描述|必须（默认）|
|-----|----|-----------|--------|
|`type`|String|Convert task identifier|Yes: `convert_segment`|
|`dataSource`|String|The datasource to search for segments|Yes|
|`interval`|Interval string|The interval in the datasource to look for segments|Yes|
|`indexSpec`|json|The compression specification for the index|Yes|
|`force`|boolean|Forces the convert task to continue even if binary versions indicate it has been updated recently (you probably want to do this)|No (false)|
|`validate`|boolean|Runs validation between the old and new segment before reporting task success|No (true)|

跟hadoop转换任务不一样,索引服务的任务会从索引服务的配置绘制它的输出路径。
### 等待任务

这些任务开始，只会在测试时被使用。语法如下：

```json
{
    "type": "noop",
    "id": <optional_task_id>,
    "interval" : <optional_segment_interval>,
    "runTime" : <optional_millis_to_sleep>,
    "firehose": <optional_firehose_to_test_connect>
}
```

锁定
---------

一旦一个霸王节点接受一项任务,在任务中数据源创建一个锁和区间指定。
任务不需要显式地释放锁,他们在任务完成时被释放。如果他们期望则任务可以早些释放锁。
任务id是惟一的命名，使用uuid或任务被创建的时间戳。
任务也“工作组”的一部分,这是一组任务,这些任务可以共享间隔锁。