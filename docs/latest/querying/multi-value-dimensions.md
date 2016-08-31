---
layout: doc_page
---
# 多值维度

Druid支持"multi-value"字符串维度。当一个输入字段包含一组值而不是一个值时(例如JSON数组或TSV领域包含一个或多个`listDelimiter` 字符)这些会生成。

这个文档描述的groupBy行为(topN也有类似的行为)多值维度的查询当他们用一个维度分组。看多值列部分在[segments](../design/segments.html#multi-value-columns) 
内部的表示细节。

## 查询多值维度


假设您有包含以下行的多值维度被称为`tags`的一个数据源。
```
{"timestamp": "2011-01-12T00:00:00.000Z", "tags": ["t1","t2","t3"]}  #row1
{"timestamp": "2011-01-13T00:00:00.000Z", "tags": ["t3","t4","t5"]}  #row2
{"timestamp": "2011-01-14T00:00:00.000Z", "tags": ["t5","t6","t7"]}  #row3
```


所有查询类型可以在多值维度过滤。过滤器独立运作每个多值维度值。例如，`"t1" 或 "t3"`过滤器将匹配row1和row2而不是row3。 `"t1" 和 "t3"`的过滤器只匹配row1。

topN和groupBy查询可以在多值组维度。当分组在一个多值的维度,_all_值从匹配行将用于生成一组值。这有可能为一个查询比多行返回更多组。
例如，dimension`tags`的topN用`"t1" OR "t3"`过滤器可以匹配row1，而且生成三组结果：`t1`, `t2`, and `t3`。
如果你仅仅需要包含匹配你过滤器的值，你可以用[filtered dimensionSpec](dimensionspecs.html#filtered-dimensionspecs)。
这也可以提高性能。
### 例子：无过滤的GroupBy 查询

查看[GroupBy querying](groupbyquery.html)更多细节。
```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "default",
      "dimension": "tags",
      "outputName": "tags"
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回以下结果。
```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t1"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t2"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t4"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t5"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t6"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t7"
    }
  }
]
```

注意原始行"exploded"到多个行和合并。
### 例子：GroupBy查询与选择查询过滤器

查看[query filters](filters.html)更多选择查询过滤器的细节。
```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "filter": {
    "type": "selector",
    "dimension": "tags",
    "value": "t3"
  },
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "default",
      "dimension": "tags",
      "outputName": "tags"
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回下面结果

```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t1"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t2"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t4"
    }
  },
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 1,
      "tags": "t5"
    }
  }
]
```

你可能会惊讶地看到包含t1，t2，t4和t5的结果。这是因为在explosion之前查询过滤器应用于行。
多值维度，选择过滤器t3将匹配row1和row2，然后explosion。
多值维度，如果有个人价值在多个值匹配查询过滤器内查询过滤器就匹配一行。
### 例子：选择查询过滤器GroupBy和“维度”属性额外的过滤器

解决上面的问题，只有行“t3”返回，你将不得不使用一个“filtered dimension spec”在如下查询中。
查看filtered dimensionSpecs部分在[dimensionSpecs](dimensionspecs.html#filtered-dimensionspecs)更多细节。
```json
{
  "queryType": "groupBy",
  "dataSource": "test",
  "intervals": [
    "1970-01-01T00:00:00.000Z/3000-01-01T00:00:00.000Z"
  ],
  "filter": {
    "type": "selector",
    "dimension": "tags",
    "value": "t3"
  },
  "granularity": {
    "type": "all"
  },
  "dimensions": [
    {
      "type": "listFiltered",
      "delegate": {
        "type": "default",
        "dimension": "tags",
        "outputName": "tags"
      },
      "values": ["t3"]
    }
  ],
  "aggregations": [
    {
      "type": "count",
      "name": "count"
    }
  ]
}
```

返回下面结果

```json
[
  {
    "timestamp": "1970-01-01T00:00:00.000Z",
    "event": {
      "count": 2,
      "tags": "t3"
    }
  }
]
```

注意，对于GroupBy查询，你可以得到和[having spec](having.html)相同的结果，但是用filtered dimensionSpec效率更高，因为能应用在查询处理管道的最低水平。
Having specs是应用于最外层groupBy查询处理的水平。