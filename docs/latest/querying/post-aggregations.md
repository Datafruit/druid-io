---
layout: doc_page
---
# Post-Aggregations
Post-aggregations是处理过程的规范，应该在Druid产生聚合值。
如果你包括post aggregation作为查询的一部分，一定要包括所有的聚合器post-aggregator要求。

这是几种可行的post-aggregators。
### 算术post-aggregators

算术post-aggregator所提供的功能适用于从左到右给定的段。字段可以是聚合器或其他post aggregators。

支持`+`， `-`，`*`， `/`，和`quotient`。
**注意**:

* `/` 如果除数是`0`，那么`/`除法都是返回`0`，不管是分式。
* `quotient`除法像普通的浮点型除法。

算术post-aggregators也可以指定一个“顺序”，它定义了顺序排序结果时产生的值(这可以用于topN查询):

-  如果没有指定顺序(或`null`)，默认使用浮点数排序。
- `numericFirst`总是先返回有限值，然后返回`NaN`，最后返回极大的值。

算术post-aggregator语法如下：
```json
postAggregation : {
  "type"  : "arithmetic",
  "name"  : <output_name>,
  "fn"    : <arithmetic_function>,
  "fields": [<post_aggregator>, <post_aggregator>, ...],
  "ordering" : <null (default), or "numericFirst">
}
```

### 字段访问器post-aggregator

这个返回的值由指定的[聚合器](../querying/aggregations.html)产生。
`fieldName`指的是给定的聚合器在[聚合](../querying/aggregations.html)查询部分输出的名称。
```json
{ "type" : "fieldAccess", "name": <output_name>, "fieldName" : <aggregator_name> }
```

### 常数 post-aggregator

常数 post-aggregator返回指定的值。
```json
{ "type"  : "constant", "name"  : <output_name>, "value" : <numerical_value> }
```

### JavaScript post-aggregator

提供JavaScript函数适用于给定的字段。字段作为参数传递给给定顺序的JavaScript函数。
```json
postAggregation : {
  "type": "javascript",
  "name": <output_name>,
  "fieldNames" : [<aggregator_name>, <aggregator_name>, ...],
  "function": <javascript function>
}
```

JavaScript aggregator例子：
```json
{
  "type": "javascript",
  "name": "absPercent",
  "fieldNames": ["delta", "total"],
  "function": "function(delta, total) { return 100 * Math.abs(delta) / total; }"
}
```
### HyperUnique 基数 post-aggregator

HyperUnique Cardinality post-aggregator是用于包装hyperUnique对象，这样它可以用于聚合。
```json
{ "type"  : "hyperUniqueCardinality", "name": <output name>, "fieldName"  : <the name field value of the hyperUnique aggregator>}
```

它可以用在实例计算如：
```json
  "aggregations" : [{
    {"type" : "count", "name" : "rows"},
    {"type" : "hyperUnique", "name" : "unique_users", "fieldName" : "uniques"}
  }],
  "postAggregations" : [{
    "type"   : "arithmetic",
    "name"   : "average_users_per_row",
    "fn"     : "/",
    "fields" : [
      { "type" : "hyperUniqueCardinality", "fieldName" : "unique_users" },
      { "type" : "fieldAccess", "name" : "rows", "fieldName" : "rows" }
    ]
  }]
```

## 用法示例

在这个示例中，我们用post aggregator计算一个简单的比例。我们先设想数据有个指标为“total”。

JSON查询格式如下：
```json
{
  ...
  "aggregations" : [
    { "type" : "count", "name" : "rows" },
    { "type" : "doubleSum", "name" : "tot", "fieldName" : "total" }
  ],
  "postAggregations" : [{
    "type"   : "arithmetic",
    "name"   : "average",
    "fn"     : "*",
    "fields" : [
       { "type"   : "arithmetic",
         "name"   : "div",
         "fn"     : "/",
         "fields" : [
           { "type" : "fieldAccess", "name" : "tot", "fieldName" : "tot" },
           { "type" : "fieldAccess", "name" : "rows", "fieldName" : "rows" }
         ]
       },
       { "type" : "constant", "name": "const", "value" : 100 }
    ]
  }]
  ...
}
```
