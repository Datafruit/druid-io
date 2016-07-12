---
layout: doc_page
---
# groupBy 查询

这些类型的查询需要groupBy查询对象，并返回一个JSON对象数组，其中每个对象代表了一个分组查询的要求。
注意：如果你只想在一段时间范围内做直接聚合，我们强烈建议用 [TimeseriesQueries](../querying/timeseriesquery.html)代替。这样性能会更好些。如果你想
按着单维度顺序groupBy，请看[TopN](../querying/topnquery.html)查询。该用例的性能也会更好些。
groupBy查询对象示例如下：
``` json
{
  "queryType": "groupBy",
  "dataSource": "sample_datasource",
  "granularity": "day",
  "dimensions": ["country", "device"],
  "limitSpec": { "type": "default", "limit": 5000, "columns": ["country", "data_transfer"] },
  "filter": {
    "type": "and",
    "fields": [
      { "type": "selector", "dimension": "carrier", "value": "AT&T" },
      { "type": "or", 
        "fields": [
          { "type": "selector", "dimension": "make", "value": "Apple" },
          { "type": "selector", "dimension": "make", "value": "Samsung" }
        ]
      }
    ]
  },
  "aggregations": [
    { "type": "longSum", "name": "total_usage", "fieldName": "user_count" },
    { "type": "doubleSum", "name": "data_transfer", "fieldName": "data_transfer" }
  ],
  "postAggregations": [
    { "type": "arithmetic",
      "name": "avg_usage",
      "fn": "/",
      "fields": [
        { "type": "fieldAccess", "fieldName": "data_transfer" },
        { "type": "fieldAccess", "fieldName": "total_usage" }
      ]
    }
  ],
  "intervals": [ "2012-01-01T00:00:00.000/2012-01-03T00:00:00.000" ],
  "having": {
  	"type": "greaterThan",
  	"aggregation": "total_usage",
  	"value": 100
  }
}
```

这里有11个groupBy查询的主要部分

|属性|描述|要求|
|--------|-----------|---------|
|queryType|This String should always be "groupBy"; this is the first thing Druid looks at to figure out how to interpret the query|yes|
|dataSource|A String or Object defining the data source to query, very similar to a table in a relational database. See [DataSource](../querying/datasource.html) for more information.|yes|
|dimensions|A JSON list of dimensions to do the groupBy over; or see [DimensionSpec](../querying/dimensionspecs.html) for ways to extract dimensions. |yes|
|limitSpec|See [LimitSpec](../querying/limitspec.html).|no|
|having|See [Having](../querying/having.html).|no|
|granularity|Defines the granularity of the query. See [Granularities](../querying/granularities.html)|yes|
|filter|See [Filters](../querying/filters.html)|no|
|aggregations|See [Aggregations](../querying/aggregations.html)|yes|
|postAggregations|See [Post Aggregations](../querying/post-aggregations.html)|no|
|intervals|A JSON Object representing ISO-8601 Intervals. This defines the time ranges to run the query over.|yes|
|context|An additional JSON Object which can be used to specify certain flags.|no|


把所有的都放在一起，上面的查询会返回 *n\*m* 数据点，上限为5000点，从`sample_datasource`表，其中n是`country`维度的基数，
m是 `device` 维度的基数，每一天都在2012-01-01和2012-01-03之间。
每个数据点都包含`total_usage`的和(long)，如果数据点的值大于100，`data_transfer` 的和(double)和 `total_usage`的结果(double)以`data_transfer`的过滤器设置为特定分组的`country` 和 `device`。
输出是这样的:
                                                                                                                                                                                                                                                                                                                  把所有的都放在一起，上面的查询会返回 *n\*m* 数据点，上限为5000点,其中n是`country`维度的基数，m是 `device` 维度的基数,每一天在2012-01-01和2012-01-01之间，从`sample_datasource`表。每个数据点都包含的(long)和`total_usage`如果数据点的值大于100,(double)和`data_transfer`和`total_usage`(double)的结果除以`data_transfer`的过滤器设置为特定分组的`country` 和 `device`。输出是这样的:的过滤器设置为特定分组的`country` 和`device`。输出是这样的:
                                                                                                                                                                                                                    
```json
[ 
  {
    "version" : "v1",
    "timestamp" : "2012-01-01T00:00:00.000Z",
    "event" : {
      "country" : <some_dim_value_one>,
      "device" : <some_dim_value_two>,
      "total_usage" : <some_value_one>,
      "data_transfer" :<some_value_two>,
      "avg_usage" : <some_avg_usage_value>
    }
  }, 
  {
    "version" : "v1",
    "timestamp" : "2012-01-01T00:00:12.000Z",
    "event" : {
      "dim1" : <some_other_dim_value_one>,
      "dim2" : <some_other_dim_value_two>,
      "sample_name1" : <some_other_value_one>,
      "sample_name2" :<some_other_value_two>,
      "avg_usage" : <some_other_avg_usage_value>
    }
  },
...
]
```

### 行为多值维度


groupBy查询可以在多值组维度分组。当分组在一个多值的维度，_all_值从匹配行将用于生成一组值。有可能一个查询比多行返回更多组。例如，groupBy与过滤维度`tags` 的`"t1" 或 "t3"`只匹配第一行，
和生成一个结果与三组:`t1`，`t2`，还有`t3`。如果你只需要包含值相匹配你的过滤器，您可以使用一个[filtered dimensionSpec](dimensionspecs.html#filtered-dimensionspecs)。这也可以提高性能。

查看[Multi-value dimensions](multi-value-dimensions.html)更多细节。