---
layout: doc_page
---
# 段元数据查询
段元数据查询返回每个段信息：
* 该段所有列的基数
* 如果他们存储在一个平面格式估计段列字节大小
* 存储在段内部的行数
* 区间段覆盖
* 段所有列的列类型
* 如果它存储在平面格式内估算总的段字节大小
* 段id

```json
{
  "queryType":"segmentMetadata",
  "dataSource":"sample_datasource",
  "intervals":["2013-01-01/2014-01-01"]
}
```

这下面是段元数据查询的几个主要部分：
|属性|描述|要求|
|--------|-----------|---------|
|queryType|This String should always be "segmentMetadata"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|intervals|A JSON Object representing ISO-8601 Intervals. This defines the time ranges to run the query over.|no|
|toInclude|A JSON Object representing what columns should be included in the result. Defaults to "all".|no|
|merge|Merge all individual segment metadata results into a single result|no|
|context|See [Context](../querying/query-context.html)|no|
|analysisTypes|A list of Strings specifying what column properties (e.g. cardinality, size) should be calculated and returned in the result. Defaults to ["cardinality", "size", "interval"]. See section [analysisTypes](#analysistypes) for more details.|no|
|lenientAggregatorMerge|If true, and if the "aggregators" analysisType is enabled, aggregators will be merged leniently. See below for details.|no|

结果格式如下：
```json
[ {
  "id" : "some_id",
  "intervals" : [ "2013-05-13T00:00:00.000Z/2013-05-14T00:00:00.000Z" ],
  "columns" : {
    "__time" : { "type" : "LONG", "hasMultipleValues" : false, "size" : 407240380, "cardinality" : null, "errorMessage" : null },
    "dim1" : { "type" : "STRING", "hasMultipleValues" : false, "size" : 100000, "cardinality" : 1944, "errorMessage" : null },
    "dim2" : { "type" : "STRING", "hasMultipleValues" : true, "size" : 100000, "cardinality" : 1504, "errorMessage" : null },
    "metric1" : { "type" : "FLOAT", "hasMultipleValues" : false, "size" : 100000, "cardinality" : null, "errorMessage" : null }
  },
  "aggregators" : {
    "metric1" : { "type" : "longSum", "name" : "metric1", "fieldName" : "metric1" }
  },
  "size" : 300000,
  "numRows" : 5000000
} ]
```

维度列有`STRING`类型
指标列将有`FLOAT` 或者 `LONG`或潜在的复杂类型的名称如`hyperUnique`的复杂指标。
Timestamp列有`LONG`类型。
如果 `errorMessage`段是非空，那你不应该相信响应的其他段，因为它们的内容是不明确的。

只有是维度（即`STRING`类型）的列才将有基数。其他列（timestamp和metric列）将基数表示为`null`。
### 间隔

如果间隔没有被指定，查询将用默认的间隔也就是跨越了一个可配置的时期结束之前最近的一段时间。
这个默认的时期长度是通过代理配置设置：
druid.query.segmentMetadata.defaultHistory
### toInclude

有三种toInclude对象类型。
#### All

语法如下：
``` json
"toInclude": { "type": "all"}
```

#### None

语法如下：
``` json
"toInclude": { "type": "none"}
```

#### List

语法如下：
``` json
"toInclude": { "type": "list", "columns": [<string list of column names>]}
```

### analysisTypes

这是一个决定返回的信息量的属性列表的列，即分析执行列。
默认情况下，将使用所有的分析类型。如果有属性是非必要的，从这个列表中忽略它将产生一个更高效的查询。
有四种列分析类型：
#### 基数

* `cardinality`在结果中将返回估计每个列基数的向下取整。只有与维度列相关。
#### size 大小

* `size`在结果中将包含估计总段字节大小正如数据存储为文本格式
#### 间隔

* `intervals`在结果中将包含与查询段相关的间隔列表
#### 聚合器

* `aggregators`在结果中将包含用于查询指标列的聚合器的列表。如果未知聚合器或者不能合并（如果启用了合并），这可能是空值。
合并可以是严格的或者是宽松的。查看 *lenientAggregatorMerge*下面有更多细节。
结果的形式是聚合列名的映射。
### lenientAggregatorMerge

如果有些部分未知的聚合器，或者两段使用不兼容的聚合器相同的列(例如longSum改为doubleSum)跨段聚合器元数据之间的冲突可能会发生。

聚合器可以严格(默认)或宽松地合并。严格的合并，如果有任何部分与未知的聚合器，或任何类型的任何冲突，合并后聚合器列表将是`null`。
宽松合并，未知聚合器的段将被忽略，聚合器之间的冲突只有在该聚合器的特定列外才是null。

特别是，宽松的合并，它可能让个人列的聚合器成为 `null`。这不会有严格的合并发生。