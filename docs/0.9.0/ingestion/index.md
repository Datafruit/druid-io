---
layout: doc_page
---

# Ingestion Spec

Druid的接入配置由3个部分组成:

```json
{
  "dataSchema" : {...},
  "ioConfig" : {...},
  "tuningConfig" : {...}
}
```

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| dataSchema | JSON Object | 指定接受数据的模式。所有接入配置可以共享同一个dataSchema。 | 是 |
| ioConfig | JSON Object | 指定数据来源和数据目的地。这个对象根据接入方法的不同而不同。 | 是 |
| tuningConfig | JSON Object | 指定如何调整各种接入参数。这个对象根据接入方法的不同而不同 | 否 |

# 数据模式

dataSchema实例如下：

```json
"dataSchema" : {
  "dataSource" : "wikipedia",
  "parser" : {
    "type" : "string",
    "parseSpec" : {
      "format" : "json",
      "timestampSpec" : {
        "column" : "timestamp",
        "format" : "auto"
      },
      "dimensionsSpec" : {
        "dimensions": ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"],
        "dimensionExclusions" : [],
        "spatialDimensions" : []
      }
    }
  },
  "metricsSpec" : [{
    "type" : "count",
    "name" : "count"
  }, {
    "type" : "doubleSum",
    "name" : "added",
    "fieldName" : "added"
  }, {
    "type" : "doubleSum",
    "name" : "deleted",
    "fieldName" : "deleted"
  }, {
    "type" : "doubleSum",
    "name" : "delta",
    "fieldName" : "delta"
  }],
  "granularitySpec" : {
    "segmentGranularity" : "DAY",
    "queryGranularity" : "NONE",
    "intervals" : [ "2013-08-31/2013-09-01" ]
  }
}
```

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| dataSource | String | 接入数据源的名称。数据源可以被想象成表 | 是 |
| parser | JSON Object | 指定如何解析摄入数据 | 是 |
| metricsSpec | JSON Object array | 一个[aggregators](../querying/aggregations.html)列表. | 是 |
| granularitySpec | JSON Object | 指定如何创建segments和roll up数据 | 是 |

## 解析器

如果不指定`type`, 解析器默认为`string`. 获取更多信息，请参考[扩展列表](../development/extensions.html).

### 字符串解析器

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| type | String | 一般设置为`string`,在hadoop indexing任务中应该设置为`hadoopyString`。 | 否 |
| parseSpec | JSON Object | 指定数据的格式, 时间戳和维度。 | 是 |

### Protobuf 解析器

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| type | String | 这项应该设定为`protobuf`. | 否 |
| parseSpec | JSON Object | 指定数据的时间戳和维度。应该是一个timeAndDims parseSpec | 是 |

### ParseSpec

ParseSpecs 的两个目的:

- 被字符串解析器用来确定接收数据行的格式（JSON,CSV,TSV）。
- 被所有解析器用来确定接收数据行的时间戳和维度

如果`format`没有被指定, 默认格式为`tsv`.

#### JSON ParseSpec

和字符串解析器用来读取JSON.

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| format | String | 应该设置为`json`。 | 否 |
| timestampSpec | JSON Object | 指定时间戳的列和格式。 | 是 |
| dimensionsSpec | JSON Object | 指定数据的维度 | 是 |
| flattenSpec | JSON Object | 指定嵌套JSON数据的扁平化配置。更多信息，请参考[扁平化JSON](./flatten-json.html)。 | 否 |

#### JSON Lowercase ParseSpec

这是JSON ParseSpec的一个特殊的变化，以小写形式接收的JSON数据的列名。如果Druid版本从0.6.x更新到0.7.x，当直接接收混合大小写列名的JSON数据而没有任何ETL处理这些列名，并且想要查询0.6.x和0.7.x创建的数据时，parseSpec参数是必须的。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| format | String | 应该设置为`jsonLowercase`。 | 是 |
| timestampSpec | JSON Object | 指定时间戳的列和格式。 | 是 |
| dimensionsSpec | JSON Object | 指定数据的维度。 | 是 |

#### CSV ParseSpec

和字符串解析器用来读取CSV。字符串用net.sf.opencsv库来解析。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| format | String | 应该设置为`csv`。 | 是 |
| timestampSpec | JSON Object | 指定时间戳的列和格式。 | 是 |
| dimensionsSpec | JSON Object | 指定数据的维度。 | 是 |
| listDelimiter | String | 多值维度的自定义分隔符。 | 否 (default == ctrl+A) |
| columns | JSON array | 指定数据的列。 | 是 |

#### TSV / Delimited ParseSpec

和字符串解析器用来读取任何被分割的不需要特殊转义的文本。默认分隔符为tab，所以可以读取TSV数据。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| format | String | 应该被设置为`tsv`. | 是 |
| timestampSpec | JSON Object | 指定时间戳的列和格式。 | 是 |
| dimensionsSpec | JSON Object | 指定数据的维度。 | 是 |
| delimiter | String | 数据值的分隔符。 | 否 (default == \t) |
| listDelimiter | String | 多值维度的自定义分隔符。 | 否 (default == ctrl+A) |
| columns | JSON String array | 定义数据的列。 | 是 |

#### TimeAndDims ParseSpec

和非字符串解析器用来提供时间戳和维度信息。非字符串解析器自己处理所有的格式化决定，而无需使用ParseSpec 。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| format | String | 应该被设置为`timeAndDims`. | 是 |
| timestampSpec | JSON Object | 指定时间戳的列和格式。 | 是 |
| dimensionsSpec | JSON Object | 指定数据的维度。 | 是 |

### TimestampSpec

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| column | String | 时间戳的列。 | 是 |
| format | String | iso, millis, posix, auto或者任何[Joda time](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html)格式. | 否 (default == 'auto' |

### DimensionsSpec

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| dimensions | JSON String array | 维度的名称。如果为空，Druid将把所有没有时间戳或者度量的列作为维度列。 | 是 |
| dimensionExclusions | JSON String array | 需排除的接入数据维度的名称 | 否 (default == [] |
| spatialDimensions | JSON Object array | [特殊维度](../development/geo.html)数组 | 否 (default == [] |

## GranularitySpec

默认为`uniform`。

### Uniform Granularity Spec

用来产生均匀间隔的segments。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| type | string | granularity spec类型。 | 否 (default == 'uniform') |
| segmentGranularity | string | 创建segments的粒度。 | 否 (default == 'DAY') |
| queryGranularity | string | 可查询结果的最小粒度和segment内部数据的粒度。例如"minute"表示按分钟聚合数据。也就是说，如果元组之间有冲突(minute(timestamp), dimensions)，将数据聚合在一起而不是分别存储每一行的值。 | 否 (default == 'NONE') |
| intervals | string | 接入的原始数据的时间间隔列表。忽略real-time接入。 | 是 for batch, 否 for real-time |

### Arbitrary Granularity Spec

用来生成任意间隔的segments（它试图创建均匀大小的段）。不支持实时处理。

| 字段 | 类型 | 描述 | 必须 |
|-------|------|-------------|----------|
| type | string | granularity spec类型。 | 否 (default == 'uniform') |
| queryGranularity | string | 可查询结果的最小粒度和segment内部数据的粒度。例如"minute"表示按分钟聚合数据。也就是说，如果元组之间有冲突(minute(timestamp), dimensions)，将数据聚合在一起而不是分别存储每一行的值。 | 否 (default == 'NONE') |
| intervals | string | 接入的原始数据的时间间隔列表。忽略real-time接入。 | 是 for batch, 否 for real-time |

# IO Config

Stream Push Ingestion: 利用Tranquility，不需要IO配置。
Stream Pull Ingestion: 参考[Stream pull ingestion](../ingestion/stream-pull.html).
Batch Ingestion: 参考[Batch ingestion](../ingestion/batch-ingestion.html)

# Tuning Config

Stream Push Ingestion: 参考[Stream push ingestion](../ingestion/stream-push.html).
Stream Pull Ingestion: 参考[Stream pull ingestion](../ingestion/stream-pull.html).
Batch Ingestion: 参考[Batch ingestion](../ingestion/batch-ingestion.html)

# Evaluating Timestamp, Dimensions and Metrics

Druid按照如下顺序编译维度，维度排除和度量：

* Dimensions里面所列出的所有列都被当作维度。
* Dimension exclusions里面的所列出的所有列都不作为维度。
* Timestamp列和metrics需要的columns/fieldNames默认不作为维度。
* 如果度量同时被列为维度，度量名称和维度名称必须不同。
