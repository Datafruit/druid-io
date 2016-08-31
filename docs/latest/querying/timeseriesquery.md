---
layout: doc_page
---
Timeseries 查询
==================

当每个对象代表timeseries 查询的一个值时，这类查询用timeseries查询对象和返回一个JSON对象数组。

timeseries查询对象示例如下：
```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "descending": "true",
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "sample_dimension1", "value": "sample_value1" },
      { "type": "or",
        "fields": [
          { "type": "selector", "dimension": "sample_dimension2", "value": "sample_value2" },
          { "type": "selector", "dimension": "sample_dimension3", "value": "sample_value3" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" },
    { "type": "doubleSum", "name": "sample_name2", "fieldName": "sample_fieldName2" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "name": "sample_name1", "fieldName": "sample_fieldName1" },
        { "type": "fieldAccess", "name": "sample_name2", "fieldName": "sample_fieldName2" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ]
}
```

timeseries查询的7个主要部分：
|属性|描述|要求|
|--------|-----------|---------|
|queryType|This String should always be "timeseries"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|descending|Whether to make descending ordered result. Default is `false`(ascending).|no|
|intervals|A JSON Object representing ISO-8601 Intervals. This defines the time ranges to run the query over.|yes|
|granularity|Defines the granularity to bucket query results. See [Granularities](../querying/granularities.html)|yes|
|filter|See [Filters](../querying/filters.html)|no|
|aggregations|See [Aggregations](../querying/aggregations.html)|yes|
|postAggregations|See [Post Aggregations](../querying/post-aggregations.html)|no|
|context|See [Context](../querying/query-context.html)|no|

把这些都放在一起，上面的查询将返回2个数据点，一个是从"sample\_datasource"表中2012-01-01到2012-01-03之间的每一天。
每个数据点sample\_fieldName1的总和（long），sample\_fieldName2的总和（double）和sample\_fieldName1的结果（double）是以sample\_fieldName2分界来过滤集合。
输出是这样的：
```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> } 
  },
  {
    "timestamp": "2012-01-02T00:00:00.000Z",
    "result": { "sample_name1": <some_value>, "sample_name2": <some_value>, "sample_divide": <some_value> }
  }
]
```

#### Zero-filling

Timeseries查询通常填空
Timeseries查询通常用零填补空的内部时间段查询。例如，如果你在2012-01-01/2012-01-04之间发出“天”的粒度timeseries查询，而且2012-01-02没有数据存在，你将获得:
```json
[
  {
    "timestamp": "2012-01-01T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  },
  {
   "timestamp": "2012-01-02T00:00:00.000Z",
   "result": { "sample_name1": 0 }
  },
  {
    "timestamp": "2012-01-03T00:00:00.000Z",
    "result": { "sample_name1": <some_value> }
  }
]
```

时间段完全地处于外部的数据间隔不是零填充。
你可以禁用所有用文本标志“skipEmptyBuckets”的零填充。在这种模式下，2012-01-02的数据点将会在结果中被省略。

这个文本标志设置查询可以是这样：
```json
{
  "queryType": "timeseries",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "aggregations": [
    { "type": "longSum", "name": "sample_name1", "fieldName": "sample_fieldName1" }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-04T00:00:00.000" ],
  "context" : {
    "skipEmptyBuckets": "true"
  }
}
```
