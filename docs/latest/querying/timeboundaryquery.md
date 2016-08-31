---
layout: doc_page
---
# Time boundary查询
Time boundary查询返回数据集的最早和最新的数据点。语法如下：
```json
{
    "queryType" : "timeBoundary",
    "dataSource": "sample_datasource",
    "bound"     : < "maxTime" | "minTime" > # optional, defaults to returning both timestamps if not set 
}
```

time boundary查询有3个主要部分：
|属性|描述|要求|
|--------|-----------|---------|
|queryType|This String should always be "timeBoundary"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|bound   | Optional, set to `maxTime` or `minTime` to return only the latest or earliest timestamp. Default to returning both if not set| no |
|context|See [Context](../querying/query-context.html)|no|

结果的格式是：
```json
[ {
  "timestamp" : "2013-05-09T18:24:00.000Z",
  "result" : {
    "minTime" : "2013-05-09T18:24:00.000Z",
    "maxTime" : "2013-05-09T18:37:00.000Z"
  }
} ]
```
