---
layout: doc_page
---
# Filter groupBy 查询结果
having clause是一个JSON对象识别哪些行从groupBy查询应该返回，通过指定条件对聚合值。
它实际上是相当于SQL子句。

Druid 支持下面几种having clauses类型。
### 数字过滤器

最简单的having clause是数字滤波器。
数字过滤器对于更复杂的boolean expressions过滤器可以用作基础。
这是一个having-clause 数字过滤的例子：
```json
{
    "type": "greaterThan",
    "aggregation": "myAggMetric",
    "value": 100
}
```

#### Equal To

equalTo filter将匹配一个指定的聚合值。
`equalTo` filter语法如下：
```json
{
    "type": "equalTo",
    "aggregation": "<aggregate_metric>",
    "value": <numeric_value>
}
```

这相当于`HAVING <aggregate> = <value>`。
#### Greater Than

greaterThan filter将匹配聚合值大于给定的值。
`greaterThan` filter语法如下：
```json
{
    "type": "greaterThan",
    "aggregation": "<aggregate_metric>",
    "value": <numeric_value>
}
```

这等同于`HAVING <aggregate> > <value>`。
#### Less Than

lessThan filter将匹配聚合值大于给定的值。
`lessThan` filter语法如下：
```json
{
    "type": "lessThan",
    "aggregation": "<aggregate_metric>",
    "value": <numeric_value>
}
```

这等同于`HAVING <aggregate> > <value>`。



### 维度选择器过滤器

#### dimSelector

dimSelector filter将匹配聚合值等于给定值。
 `dimSelector` filter语法如下：
```json
{
    "type": "dimSelector",
    "dimension": "<dimension>",
    "value": <dimension_value>
}
```


### Logical expression filters

#### AND

AND filter语法如下：
```json
{
    "type": "and",
    "havingSpecs": [<having clause>, <having clause>, ...]
}
```

`havingSpecs` 的having clauses可以是这个页面定义的任何having clauses。
#### OR

OR filter的语法如下：
```json
{
    "type": "or",
    "havingSpecs": [<having clause>, <having clause>, ...]
}
```

`havingSpecs` 的having clauses可以是这个页面定义的任何having clauses。

#### NOT

NOT filter语法如下：
```json
{
    "type": "not",
    "havingSpec": <having clause>
}
```

`havingSpecs` 的having clauses可以是这个页面定义的任何having clauses。