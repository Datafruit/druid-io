---
layout: doc_page
---
# 聚合


Aggregations可以在ingestion时作为提供ingestion spec的总结数据之前进入Druid，还可以指定为查询时多查询的一部分。

有效的aggregations是：
### 计数聚合

`count`匹配filters的Druid行数

```json
{ "type" : "count", "name" : <output_name> }
```

请注意计数聚合Druid行数,这并不总是反映原始事件的数量被ingested。
这是因为Druid在ingestion时rolls up数据。为了计算ingested行数的数据,包括在ingestion时的count aggregator,和查询时的一个longSum聚合器。

### Sum aggregators 

#### `longSum` aggregator longSum 

计算一个64-bit,签署了integer的总值

```json
{ "type" : "longSum", "name" : <output_name>, "fieldName" : <metric_name> }
```

`name` –  为总值输出的名称
`fieldName` –  指定列名的和

#### `doubleSum` aggregator 

计算64-bit floating值的和。类似 `longSum`
```json
{ "type" : "doubleSum", "name" : <output_name>, "fieldName" : <metric_name> }
```

### Min / Max aggregators 

#### `doubleMin` aggregator 

`doubleMin` 计算所有指定列值和Double.POSITIVE_INFINITY的最小值

```json
{ "type" : "doubleMin", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `doubleMax` aggregator 

`doubleMax`计算所有指定列值和Double.POSITIVE_INFINITY的最大值

```json
{ "type" : "doubleMax", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `longMin` aggregator

`longMin`计算所有指定列值和Long.MAX_VALUE的最小值

```json
{ "type" : "longMin", "name" : <output_name>, "fieldName" : <metric_name> }
```

#### `longMax` aggregator

`longMax` 计算所有指定列值和Long.MAX_VALUE的最大值
```json
{ "type" : "longMax", "name" : <output_name>, "fieldName" : <metric_name> }
```

### JavaScript aggregator

计算在一组列(包括metrics和dimensions)的任意JavaScript函数。
所有javascript函数必须返回numerical values。
JavaScript aggregators比本地Java aggregators慢得多,如果性能是至关重要的,你应该实现你的本地Java aggregator函数。

```json
{ "type": "javascript",
  "name": "<output_name>",
  "fieldNames"  : [ <column1>, <column2>, ... ],
  "fnAggregate" : "function(current, column1, column2, ...) {
                     <updates partial aggregate (current) based on the current row values>
                     return <updated partial aggregate>
                   }",
  "fnCombine"   : "function(partialA, partialB) { return <combined partial results>; }",
  "fnReset"     : "function()                   { return <initial value>; }"
}
```

**例子** 

```json
{
  "type": "javascript",
  "name": "sum(log(x)*y) + 10",
  "fieldNames": ["x", "y"],
  "fnAggregate" : "function(current, a, b)      { return current + (Math.log(a) * b); }",
  "fnCombine"   : "function(partialA, partialB) { return partialA + partialB; }",
  "fnReset"     : "function()                   { return 10; }"
}
```

javascript聚合器是以快速原型的特征而被推荐的。该聚合器将在生产中要比使用本地Java聚合器慢得多。
## 近似聚合

### 基数聚合器


计算一组Druid的基数,用HyperLogLog估计基数。请注意,这个聚合器的速度将大大慢于hyperUnique聚合器的索引列。该聚合器还运行在一个维度列，
这意味着不能从数据集中删除字符串维度而去改善汇总。一般来说,如果你不关心个人价值维度，我们强烈建议使用hyperUnique聚合器而不是基数的聚合器。

```json
{
  "type": "cardinality",
  "name": "<output_name>",
  "fieldNames": [ <dimension1>, <dimension2>, ... ],
  "byRow": <false | true> # (optional, defaults to false)
}
```

####  基数的价值

当byRow设置为false(默认)它是计算所有给定维度的一组所有维度值的聚合。

* 对于单维度，这相当于

```sql
SELECT COUNT(DISTINCT(dimension)) FROM <datasource>
```

* 对于多维度，这相当类似于

```sql
SELECT COUNT(DISTINCT(value)) FROM (
  SELECT dim_1 as value FROM <datasource>
  UNION
  SELECT dim_2 as value FROM <datasource>
  UNION
  SELECT dim_3 as value FROM <datasource>
)
```

#### Cardinality by row  

当设置byRow为true时，它通过行来计算基数，即不同的维度组合的基数
```sql
SELECT COUNT(*) FROM ( SELECT DIM1, DIM2, DIM3 FROM <datasource> GROUP BY DIM1, DIM2, DIM3 )
```

**例子** 


人们所生活或出生的地方决定国家的数量。
```json
{
  "type": "cardinality",
  "name": "distinct_countries",
  "fieldNames": [ "coutry_of_origin", "country_of_residence" ]
}
```


决定不同的人的数量（即姓和名的组合）。
```json
{
  "type": "cardinality",
  "name": "distinct_people",
  "fieldNames": [ "first_name", "last_name" ],
  "byRow" : true
}
```

### HyperUnique聚合器


使用[HyperLogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)来计算已经聚合在索引时作为“hyperUnique”指标的单维度的估计基数
```json
{ "type" : "hyperUnique", "name" : <output_name>, "fieldName" : <metric_name> }
```


更多近似聚合器，请看[theta sketches](../development/extensions-core/datasketches-aggregators.html).
## 各种各样的聚合

### 过滤聚合器

过滤聚合器包装着所有给定的聚合器,但只有聚集给定维度筛选匹配的值。
这同时可以计算被过滤和没被聚合的结果，如果没有多个查询事件那就可以用这两个结果作为聚合的一部分。
*局限：* 过滤聚合器现在只支持'or'，'and'，'selector'， 'not' 和 'Extraction'，即匹配一个或多个维度而不是单个值。

*注意:* 如果只是要过滤结果，就考虑在查询本身过滤,它将快得多,因为它不需要扫描所有的数据。


```json
{
  "type" : "filtered",
  "filter" : {
    "type" : "selector",
    "dimension" : <dimension>,
    "value" : <dimension value>
  }
  "aggregator" : <aggregation>
}
```
