---
layout: doc_page
---

# 摄取规格

Druid摄取规格由3部分组成：
```json
{
  "dataSchema" : {...},
  "ioConfig" : {...},
  "tuningConfig" : {...}
}
```

 |字段|类型|描述|要求|
|-------|------|-------------|----------|
| dataSchema | JSON Object | Specifies the the schema of the incoming data. All ingestion specs can share the same dataSchema. | yes |
| ioConfig | JSON Object | Specifies where the data is coming from and where the data is going. This object will vary with the ingestion method. | yes |
| tuningConfig | JSON Object | Specifies how to tune various ingestion parameters. This object will vary with the ingestion method. | no |

# DataSchema

dataschema示例如下：

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

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| dataSource | String | The name of the ingested datasource. Datasources can be thought of as tables. | yes |
| parser | JSON Object | Specifies how ingested data can be parsed. | yes |
| metricsSpec | JSON Object array | A list of [aggregators](../querying/aggregations.html). | yes |
| granularitySpec | JSON Object | Specifies how to create segments and roll up data. | yes |

## 解析

如果不包括 `type`，解析器默认为 `string`。对于另外的数据格式，请查阅我们的[扩展列表](../development/extensions.html)。
### 字符串解析

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `string` in general, or `hadoopyString` when used in a Hadoop indexing job. | no |
| parseSpec | JSON Object | Specifies the format, timestamp, and dimensions of the data. | yes |

### Protobuf 解析

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `protobuf`. | no |
| parseSpec | JSON Object | Specifies the timestamp and dimensions of the data. Should be a timeAndDims parseSpec. | yes |

### 解析

解析规格的两个目的：

- 字符串解析器使用它们来确定传入行的格式（即JSON，CSV，ISV）。
- 所有的解析器使用它们来确定timestamp和传入行的维度。

如果不包括 `format` ，解析规格默认`tsv`。

#### JSON 解析

使用字符串解析器加载JSON。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| format | String | This should say `json`. | no |
| timestampSpec | JSON Object | Specifies the column and format of the timestamp. | yes |
| dimensionsSpec | JSON Object | Specifies the dimensions of the data. | yes |
| flattenSpec | JSON Object | Specifies flattening configuration for nested JSON data. See [Flattening JSON](./flatten-json.html) for more info. | no |

#### JSON小写解析

这是一个特殊的JSON解析规格变异，所有传入的JSON数据的列名用小写字母。如果你把Druid 0.6.x更新到Druid 0.7.x，这个解析规格就是必须的，是直接用大小写命名摄取JSON的列名称，
没有任何的ETL适应这些小写的列名，而且可以查询包括你使用 0.6.x和 0.7.x创建的数据。


|字段|类型|描述|要求|
|-------|------|-------------|----------|
| format | String | This should say `jsonLowercase`. | yes |
| timestampSpec | JSON Object | Specifies the column and format of the timestamp. | yes |
| dimensionsSpec | JSON Object | Specifies the dimensions of the data. | yes |

#### CSV 解析规格

用字符串解析器加载CSV。字符串是使用net.sf.opencsv库解析的。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| format | String | This should say `csv`. | yes |
| timestampSpec | JSON Object | Specifies the column and format of the timestamp. | yes |
| dimensionsSpec | JSON Object | Specifies the dimensions of the data. | yes |
| listDelimiter | String | A custom delimiter for multi-value dimensions. | no (default == ctrl+A) |
| columns | JSON array | Specifies the columns of the data. | yes |

#### TSV / 分隔解析规格

使用字符串解析器加载任何分隔的文本不会要求特殊的转义。默认情况下，delimiter是一个标签，所以这个将会加载ISV。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| format | String | This should say `tsv`. | yes |
| timestampSpec | JSON Object | Specifies the column and format of the timestamp. | yes |
| dimensionsSpec | JSON Object | Specifies the dimensions of the data. | yes |
| delimiter | String | A custom delimiter for data values. | no (default == \t) |
| listDelimiter | String | A custom delimiter for multi-value dimensions. | no (default == ctrl+A) |
| columns | JSON String array | Specifies the columns of the data. | yes |

#### TimeAndDims解析规格

使用non-String解析器提供给它们timestamp和dimensions信息。non-String解析器由它们自己决定处理的格式，不需要使用解析规格。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| format | String | This should say `timeAndDims`. | yes |
| timestampSpec | JSON Object | Specifies the column and format of the timestamp. | yes |
| dimensionsSpec | JSON Object | Specifies the dimensions of the data. | yes |

### 时间戳规格

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| column | String | The column of the timestamp. | yes |
| format | String | iso, millis, posix, auto or any [Joda time](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html) format. | no (default == 'auto' |

### 维度规格

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| dimensions | JSON String array | The names of the dimensions. If this is an empty array, Druid will treat all columns that are not timestamp or metric columns as dimension columns. | yes |
| dimensionExclusions | JSON String array | The names of dimensions to exclude from ingestion. | no (default == [] |
| spatialDimensions | JSON Object array | An array of [spatial dimensions](../development/geo.html) | no (default == [] |

## 粒度规格

默认的粒度规格是`uniform`。

### 统一的粒度规格

这个规格用来生成统一的时间间隔的Segment。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | string | The type of granularity spec. | no (default == 'uniform') |
| segmentGranularity | string | The granularity to create segments at. | no (default == 'DAY') |
| queryGranularity | string | The minimum granularity to be able to query results at and the granularity of the data inside the segment. E.g. a value of "minute" will mean that data is aggregated at minutely granularity. That is, if there are collisions in the tuple (minute(timestamp), dimensions), then it will aggregate values together using the aggregators instead of storing individual rows. | no (default == 'NONE') |
| intervals | string | A list of intervals for the raw data being ingested. Ignored for real-time ingestion. | yes for batch, no for real-time |

### 任意的粒度规格

这个规格用来生成任意区间的Segment（它试着创建均匀大小的Segments）。这个规格不支持real-time处理。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | string | The type of granularity spec. | no (default == 'uniform') |
| queryGranularity | string | The minimum granularity to be able to query results at and the granularity of the data inside the segment. E.g. a value of "minute" will mean that data is aggregated at minutely granularity. That is, if there are collisions in the tuple (minute(timestamp), dimensions), then it will aggregate values together using the aggregators instead of storing individual rows. | no (default == 'NONE') |
| intervals | string | A list of intervals for the raw data being ingested. Ignored for real-time ingestion. | yes for batch, no for real-time |

# IO配置

流推动摄取：流使用Tranguility推动摄取不需要一个IO配置。
流拉入摄取：查阅 [流拉动摄取](../ingestion/stream-pull.html)。
批量摄取：查阅 [批量获取](../ingestion/batch-ingestion.html)

# 优化配置

流推动摄取：查阅[流推动摄取](../ingestion/stream-push.html)。
流拉入摄取：查阅 [流拉动摄取](../ingestion/stream-pull.html)。
批量摄取：查阅 [批量获取](../ingestion/batch-ingestion.html)。

# 评估时间戳，维度和指标

Druid对维度、维度除外和指标的说明分别如下：:

* 任何维度中的列都被看成是维度。
* 任何维度以外中的列都不被认为是维度。
* 指标所需的timestamp列和列/字段名默认情况下被排除在外。
* 如果一个指标也是列为维度，那么该指标必须有个与维度名称不同的名称。