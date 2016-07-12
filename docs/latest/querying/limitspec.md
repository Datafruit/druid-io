---
layout: doc_page
---
# Sort groupBy 查询结果
limitSpec字段提供了功能分类和限制groupBy查询的结果集。如果你以一个单维度分组和以单一指标排序，我们强烈建议使用[TopN Queries](../querying/topnquery.html)。性能会更好。可用的选项是:

### DefaultLimitSpec

默认限制规范需要限制和列的列表做orderBy操作结束。语法是:
```json 
{
    "type"    : "default",
    "limit"   : <integer_value>,
    "columns" : [list of OrderByColumnSpec],
}
```

#### OrderByColumnSpec

OrderByColumnSpecs根据操作排序指示怎么做。每个order by条件可以是`jsonString`或下面形式的示意:
```json 
{
    "dimension" : "<Any dimension or metric name>",
    "direction" : <"ascending"|"descending">
}
```

如果只提供维度(如一个JSON字符串），默认order-by是递增。