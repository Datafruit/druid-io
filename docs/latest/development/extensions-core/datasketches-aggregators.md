---
layout: doc_page
---

## DataSketches 聚合

Druid聚合器基于[datasketches](http://datasketches.github.io/)库。注意草图算法是近似的；
在datasketches文档的“Accuracy”章节查阅更多细节。
在摄取时，这个聚合器创建存储在druid段的θ草图对象。

从逻辑上讲,θ草图对象可以被认为是一组数据结构。在查询时,读取和聚合草图(联合)在一起。
在最后，默认情况下，你收到草图对象中的唯一条目对象的估计数量。
而且，你可以使用post aggregator把同一行中的列并集，交集或不同草图。
注意你可以在列中使用`thetaSketch`聚合器，它将不会使用相同的摄取，它将返回列的粒度估算值。
这建议我们在摄取时使用它也可以让查询快些。

为了使用datasketch聚合，请确保你的配置文件中[包含](../../operations/including-extensions.html)扩展：

```
druid.extensions.loadList=["druid-datasketches"]
```

### 聚合

```json
{
  "type" : "thetaSketch",
  "name" : <output_name>,
  "fieldName" : <metric_name>,  
  "isInputThetaSketch": false,
  "size": 16384
 }
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This String should always be "thetaSketch"|yes|
|name|A String for the output (result) name of the calculation.|yes|
|fieldName|A String for the name of the aggregator used at ingestion time.|yes|
|isInputThetaSketch|This should only be used at indexing time if your input data contains theta sketch objects. This would be the case if you use datasketches library outside of Druid, say with Pig/Hive, to produce the data that you are ingesting into Druid |no, defaults to false|
|size|Must be a power of 2. Internally, size refers to the maximum number of entries sketch object will retain. Higher size means higher accuracy but more space to store sketches. Note that after you index with a particular size, druid will persist sketch in segments and you will use size greater or equal to that at query time. See [theta-size](http://datasketches.github.io/docs/ThetaSize.html) for details. In general, We recommend just sticking to default size. |no, defaults to 16384|

### post 聚合器

#### 草图估计量

```json
{
  "type"  : "thetaSketchEstimate",
  "name": <output name>,
  "field"  : <post aggregator of type fieldAccess that refers to a thetaSketch aggregator or that of type thetaSketchSetOp>
}
```

#### 草图操作

```json
{
  "type"  : "thetaSketchSetOp",
  "name": <output name>,
  "func": <UNION|INTERSECT|NOT>,
  "fields"  : <the name field value of the thetaSketch aggregators>,
  "size": <16384 by default, must be max of size from sketches in fields input>
}
```

### 示例

假设，你有一个数据组包含(timestamp, product, user_id)。你要解决如下问题

多少个用户访问产品A？
多少个用户访问了产品A和产品B？

为了解决上面的问题，你应该使用下面的聚合器索引你的数据。

```json
{ "type": "thetaSketch", "name": "user_id_sketch", "fieldName": "user_id" }
```

然后，有多少个用户访问了产品A?示例如下

```json
{
  "queryType": "groupBy",
  "dataSource": "test_datasource",
  "granularity": "ALL",
  "dimensions": [],
  "aggregations": [
    { "type": "thetaSketch", "name": "unique_users", "fieldName": "user_id_sketch" }
  ],
  "filter": { "type": "selector", "dimension": "product", "value": "A" },
  "intervals": [ "2014-10-19T00:00:00.000Z/2014-10-22T00:00:00.000Z" ]
}
```

示例查询，有多少个用户访问了产品A和产品B？

```json
{
  "queryType": "groupBy",
  "dataSource": "test_datasource",
  "granularity": "ALL",
  "dimensions": [],
  "filter": {
    "type": "or",
    "fields": [
      {"type": "selector", "dimension": "product", "value": "A"},
      {"type": "selector", "dimension": "product", "value": "B"}
    ]
  },
  "aggregations": [
    {
      "type" : "filtered",
      "filter" : {
        "type" : "selector",
        "dimension" : "product",
        "value" : "A"
      },
      "aggregator" :     {
        "type": "thetaSketch", "name": "A_unique_users", "fieldName": "user_id_sketch"
      }
    },
    {
      "type" : "filtered",
      "filter" : {
        "type" : "selector",
        "dimension" : "product",
        "value" : "B"
      },
      "aggregator" :     {
        "type": "thetaSketch", "name": "B_unique_users", "fieldName": "user_id_sketch"
      }
    }
  ],
  "postAggregations": [
    {
      "type": "thetaSketchEstimate",
      "name": "final_unique_users",
      "field":
      {
        "type": "thetaSketchSetOp",
        "name": "final_unique_users_sketch",
        "func": "INTERSECT",
        "fields": [
          {
            "type": "fieldAccess",
            "fieldName": "A_unique_users"
          },
          {
            "type": "fieldAccess",
            "fieldName": "B_unique_users"
          }
        ]
      }
    }
  ],
  "intervals": [
    "2014-10-19T00:00:00.000Z/2014-10-22T00:00:00.000Z"
  ]
}
```
