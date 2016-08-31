---
layout: doc_page
---
# 数据源的元数据查询
数据源的元数据查询返回数据源的元数据信息。这些查询返回信息:

* 数据源的最新摄取事件的时间标记。这是没有任何考虑rollup的摄取事件

这些查询语法为：

```json
{
    "queryType" : "dataSourceMetadata",
    "dataSource": "sample_datasource"
}
```

主要有2部分数据源的元数据查询:
|性能|描述|要求
|--------|-----------|---------|
|queryType|This String should always be "dataSourceMetadata"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|context|See [Context](../querying/query-context.html)|no|

结果的格式：

```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "maxIngestedEventTime" : "2013-05-09T18:24:09.007Z",
  }
} ]
```
