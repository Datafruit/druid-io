---
layout: doc_page
---

# JSON 压缩规格

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| useFieldDiscovery | Boolean | If true, interpret all fields with singular values (not a map or list) and flat lists (lists of singular values) at the root level as columns. | no (default == true) |
| fields | JSON Object array | Specifies the fields of interest and how they are accessed | no (default == []) |

定义JSON压缩规格允许嵌套JSON字段在摄取期间压缩。只有JSON解析规格支持压缩。
'fields'是一个JSON对象列，描述字段名称和如何访问字段：
## JSON字段规格

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | Type of the field, "root" or "nested". | yes |
| name | String | This string will be used as the column name when the data has been ingested.  | yes |
| expr | String | Defines an expression for accessing the field within the JSON object, using [JsonPath](https://github.com/jayway/JsonPath) notation. | yes |

JSON支持以下事件格式：
```json
{
 "timestamp": "2015-09-12T12:10:53.155Z",
 "dim1": "qwerty",
 "dim2": "asdf",
 "dim3": "zxcv",
 "ignore_me": "ignore this",
 "metrica": 9999,
 "foo": {"bar": "abc"},
 "foo.bar": "def",
 "nestmet": {"val": 42},
 "hello": [1.0, 2.0, 3.0, 4.0, 5.0],
 "mixarray": [1.0, 2.0, 3.0, 4.0, {"last": 5}],
 "world": [{"hey": "there"}, {"tree": "apple"}],
 "thing": {"food": ["sandwich", "pizza"]}
}
```

“metrica”列是一个long指标列，“hello”是一组double指标，“nestmet.val”是一个嵌套的long指标。其他列都是维度。

为了压缩这个JSON，解析规格可以定义如下：

```json
"parseSpec": {
  "format": "json",
  "flattenSpec": {
    "useFieldDiscovery": true,
    "fields": [
      {
        "type": "root",
        "name": "dim1",
        "expr": "dim1"
      },
      "dim2",
      {
        "type": "nested",
        "name": "foo.bar",
        "expr": "$.foo.bar"
      },
      {
        "type": "root",
        "name": "root-foo.bar",
        "expr": "foo.bar"
      },
      {
        "type": "nested",
        "name": "nested-metric",
        "expr": "$.nestmet.val"
      },
      {
        "type": "nested",
        "name": "hello-0",
        "expr": "$.hello[0]"
      },
      {
        "type": "nested",
        "name": "hello-4",
        "expr": "$.hello[4]"
      },
      {
        "type": "nested",
        "name": "world-hey",
        "expr": "$.world.hey"
      },
      {
        "type": "nested",
        "name": "worldtree",
        "expr": "$.world.tree"
      },
      {
        "type": "nested",
        "name": "first-food",
        "expr": "$.thing.food[0]"
      },
      {
        "type": "nested",
        "name": "second-food",
        "expr": "$.thing.food[1]"
      }
    ]
  },
  "dimensionsSpec" : {
   "dimensions" : [],
   "dimensionsExclusions": ["ignore_me"]
  },
  "timestampSpec" : {
   "format" : "auto",
   "column" : "timestamp"
  }
}
```

字段 "dim3", "ignore_me", 和 "metrica"将会自动地被发现，因为‘useFieldDiscovery’是true，所以它们从字段规格列表中被省略了。

"ignore_me"将会自动被发现但是排除通过dimensionsExclusions指定.
聚合器应该使用压缩规格中定义的列的名称。使用上面的示例：
```json
"metricsSpec" : [ 
{
  "type" : "longSum",
  "name" : "Nested Metric Value",
  "fieldName" : "nested-metric"
}, 
{
  "type" : "doubleSum",
  "name" : "Hello Index #0",
  "fieldName" : "hello-0"
},
{
  "type" : "longSum",
  "name" : "metrica",
  "fieldName" : "metrica"
}
]
```

注意：

* 为了方便，当定义一个root-level字段时，可以只定义字段名，用字符串表示，如上面“dim2”所示。

* 启用'useFieldDiscovery'只会在根级别自动检测单个值的字段(不是一个映射或列表),以及字段指的是单个值的列表。
 在上面的示例中, “dim1”、“dim2”、“dim3”、“ignore_me”、“metrica”,和“foo.bar”(在根)将被自动检测为列。
“hello”字段是一个double列而且能自动被发现， 但是注意,如摄入个体成员作为单独的字段列表。“world”字段必须明确定义，因为它最后的值是一个映射。
“mixarray”字段,类似于“hello”, 也必须明确定义,因为它最后的值是一个映射。

* 不允许定义重复的字段,会抛出一个异常。
* 如果字段自动发现是启动的，所有名字相同的字段如一个已经在字段规格定义，就将跳过而不会加载两次。
* JSON输入必须是root底下的一个JSON对象，而不是一个数组。如{"valid": "true"} 和{"valid":[1,2,3]} 是支持的，而 [{"invalid": "true"}]和 [1,2,3]是不支持的。