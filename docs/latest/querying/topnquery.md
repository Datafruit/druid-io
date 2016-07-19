---
layout: doc_page
---
TopN queries
==================

TopN查询针对给定的维度，根据某些规则返回一组有序的结果集。从概念上来说，它可以被近似看做带有[排序](../querying/limitspec.html)特性的单维度的[GroupBy查询]。TopN在这种场景下比GroupBy高效得多，且资源消耗更少。这种类型的查询使用一个topN查询对象，放回一组JSON对象，其中每个对象表示一个topN查询请求的值。

TopN近似于在每个节点排出它们的K个top结果，然后仅返回这K个结果至broker节点。K，在Druid中的默认值为`max(1000, threshold)`。在实践中，这意味着如果你请求top 1000个有序项，前900个项的准确率为100%，而无法保证这之后的结果项的顺序。通过调整这个阈值，可以是TopN的结果更加准确。

一个典型的topN查询对象如下所示:

```json
{
  "queryType": "topN",
  "dataSource": "sample_data",
  "dimension": "sample_dim",
  "threshold": 5,
  "metric": "count",
  "granularity": "all",
  "filter": {
    "type": "and",
    "fields": [
      {
        "type": "selector",
        "dimension": "dim1",
        "value": "some_value"
      },
      {
        "type": "selector",
        "dimension": "dim2",
        "value": "some_other_val"
      }
    ]
  },
  "aggregations": [
    {
      "type": "longSum",
      "name": "count",
      "fieldName": "count"
    },
    {
      "type": "doubleSum",
      "name": "some_metric",
      "fieldName": "some_metric"
    }
  ],
  "postAggregations": [
    {
      "type": "arithmetic",
      "name": "sample_divide",
      "fn": "/",
      "fields": [
        {
          "type": "fieldAccess",
          "name": "some_metric",
          "fieldName": "some_metric"
        },
        {
          "type": "fieldAccess",
          "name": "count",
          "fieldName": "count"
        }
      ]
    }
  ],
  "intervals": [
    "2013-08-31T00:00:00.000/2013-09-03T00:00:00.000"
  ]
}
```
11个topN查询部分。

|属性|描述|要求|
|--------|-----------|---------|
|queryType|This String should always be "topN"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|intervals|A JSON Object representing ISO-8601 Intervals. This defines the time ranges to run the query over.|yes|
|granularity|Defines the granularity to bucket query results. See [Granularities](../querying/granularities.html)|yes|
|filter|See [Filters](../querying/filters.html)|no|
|aggregations|See [Aggregations](../querying/aggregations.html)|yes|
|postAggregations|See [Post Aggregations](../querying/post-aggregations.html)|no|
|dimension|A String or JSON object defining the dimension that you want the top taken for. For more info, see [DimensionSpecs](../querying/dimensionspecs.html)|yes|
|threshold|An integer defining the N in the topN (i.e. how many results you want in the top list)|yes|
|metric|A String or JSON object specifying the metric to sort by for the top list. For more info, see [TopNMetricSpec](../querying/topnmetricspec.html).|yes|
|context|See [Context](../querying/query-context.html)|no|

请注意JSON对象文本对topN查询也是可行的而且应该用与timeseries事件相同的警告。
结果的格式应该像这样：

```json
[
  {
    "timestamp": "2013-08-31T00:00:00.000Z",
    "result": [
      {
        "dim1": "dim1_val",
        "count": 111,
        "some_metrics": 10669,
        "average": 96.11711711711712
      },
      {
        "dim1": "another_dim1_val",
        "count": 88,
        "some_metrics": 28344,
        "average": 322.09090909090907
      },
      {
        "dim1": "dim1_val3",
        "count": 70,
        "some_metrics": 871,
        "average": 12.442857142857143
      },
      {
        "dim1": "dim1_val4",
        "count": 62,
        "some_metrics": 815,
        "average": 13.14516129032258
      },
      {
        "dim1": "dim1_val5",
        "count": 60,
        "some_metrics": 2787,
        "average": 46.45
      }
    ]
  }
]
```

### 多值维度的行为

topN查询可以在多值维度上分组。当在多值维度上分组时，从匹配行得到的_all_值为每个值生成一个组。
这让查询返回比行更多的组。比如，一个过滤 `"t1" 或者 "t3"` 的维度 `tags` 的topN只匹配row1，和生成3组结果：`t1`， `t2`，和`t3`。如果你只需要匹配你的筛选条件的值，你可以用一个[filtered dimensionSpec](dimensionspecs.html#filtered-dimensionspecs)。这个也可以提高性能。

查看[多值维度](multi-value-dimensions.html) 更多细节。
### Aliasing

当前TopN算法是一种近似算法。每个段的前1000名本地结果返回合并以确定全球topN。就这样而言，topN算法是等级和结果的近似。近似结果*只有超过1000个dim 值时才应用*。一个维度少于1000个唯一的维度值的topN可以考虑等级精确和聚合精确。
临界值可以通过需要重启生效或在查询上下文中设置`minTopNThreshold`让每个查询生效的服务器参数`druid.query.topN.minTopNThreshold`从默认的1000修改。

如果你想要high cardinality的前100个，维度按照一些low-cardinality排序均匀地分布，均匀地分布维度，你可能会得到聚合时缺失的数据。换句话说，最好的topN用例是当你有信心把整体结果统一在顶部时。例如，如果一个特殊的网址ID是一些每天每个小时排名前10的指标，那么它可能会在topN准确多天。但是如果一个网址对于给定的时间内很少排在前1000，但是整个查询粒度在前500（例如：一个高度统一的网站交通混合在网址和高度周期性数据的数据集），然后一个前500查询可能不会有特殊的网站的排名，也可能对特殊网站的聚合并不精准。
在继续这部分之前，请考虑你是否需要精准的结果。得到精准的结果是一个资源集密型的过程。对于绝大多数“有用的”数据结果，一种近似topN算法已提供足够的精度。

用户希望得到一个随着维度超过1000唯一值，应该产生一个分组查询和自己对结果进行排序的*精确等级和精确聚合*的topN。这对于high-cardinality维度是需要大量计算的。用户可以接受*近似等级*topN维度超过1000个唯一值，但是要求*精确聚合*可以产生两个查询。一个得到近似topN维度值，另一个维度选择过滤器topN只使用第一个topN结果。
#### 查询示例1:

```json
{
    "aggregations": [
             {
                 "fieldName": "L_QUANTITY_longSum",
                 "name": "L_QUANTITY_",
                 "type": "longSum"
             }
    ],
    "dataSource": "tpch_year",
    "dimension":"l_orderkey",
    "granularity": "all",
    "intervals": [
        "1900-01-09T00:00:00.000Z/2992-01-10T00:00:00.000Z"
    ],
    "metric": "L_QUANTITY_",
    "queryType": "topN",
    "threshold": 2
}
```

#### 查询示例2:

```json
{
    "aggregations": [
             {
                 "fieldName": "L_TAX_doubleSum",
                 "name": "L_TAX_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_DISCOUNT_doubleSum",
                 "name": "L_DISCOUNT_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_EXTENDEDPRICE_doubleSum",
                 "name": "L_EXTENDEDPRICE_",
                 "type": "doubleSum"
             },
             {
                 "fieldName": "L_QUANTITY_longSum",
                 "name": "L_QUANTITY_",
                 "type": "longSum"
             },
             {
                 "name": "count",
                 "type": "count"
             }
    ],
    "dataSource": "tpch_year",
    "dimension":"l_orderkey",
    "filter": {
        "fields": [
            {
                "dimension": "l_orderkey",
                "type": "selector",
                "value": "103136"
            },
            {
                "dimension": "l_orderkey",
                "type": "selector",
                "value": "1648672"
            }
        ],
        "type": "or"
    },
    "granularity": "all",
    "intervals": [
        "1900-01-09T00:00:00.000Z/2992-01-10T00:00:00.000Z"
    ],
    "metric": "L_QUANTITY_",
    "queryType": "topN",
    "threshold": 2
}
```
