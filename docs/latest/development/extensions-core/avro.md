---
layout: doc_page
---

# Avro

这个扩展能够使Druid摄取和理解Apache Avro数据格式。
确保[包含](../../operations/including-extensions.html) `druid-avro-extensions`作为一个扩展。
### Avro 流解析器

这个是针对流/实时摄取的。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `avro_stream`. | no |
| avroBytesDecoder | JSON Object | Specifies how to decode bytes to Avro record. | yes |
| parseSpec | JSON Object | Specifies the timestamp and dimensions of the data. Should be a timeAndDims parseSpec. | yes |

例如，使用Avro流解析模式回购Avro字节解码器：

```json
"parser" : {
  "type" : "avro_stream",
  "avroBytesDecoder" : {
    "type" : "schema_repo",
    "subjectAndIdConverter" : {
      "type" : "avro_1124",
      "topic" : "${YOUR_TOPIC}"
    },
    "schemaRepository" : {
      "type" : "avro_1124_rest_client",
      "url" : "${YOUR_SCHEMA_REPO_END_POINT}",
    }
  },
  "parseSpec" : {
    "type": "timeAndDims",
    "timestampSpec": <standard timestampSpec>,
    "dimensionsSpec": <standard dimensionsSpec>
  }
}
```

#### Avro 字节解码器

如果没有包含`type`，avroByteaDecoder默认为 `schema_repo`。

##### 模式回购基于Avro字节解码器

这个Avro字节解码器首先从输入信息字节提取`subject`和`id`，然后使用他们从字节查找解码Avro记录的Avro模式
更多细节可以查看[模式回购](https://github.com/schema-repo/schema-repo)和[AVRO-1124](https://issues.apache.org/jira/browse/AVRO-1124)
你将需要一个http服务器如模式回购去持有avro模式。对于在信息生产方面的模式注册，你可以参考 `io.druid.data.input.AvroStreamInputRowParserTest#testParse()`。

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `schema_repo`. | no |
| subjectAndIdConverter | JSON Object | Specifies the how to extract subject and id from message bytes. | yes |
| schemaRepository | JSON Object | Specifies the how to lookup Avro schema from subject and id. | yes |

##### Avro-1124 主题和Id转换器

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `avro_1124`. | no |
| topic | String | Specifies the topic of your kafka stream. | yes |


##### Avro-1124 模式库

|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `avro_1124_rest_client`. | no |
| url | String | Specifies the endpoint url of your Avro-1124 schema repository. | yes |

### Avro Hadoop 解析器

这个是针对使用HadoopDruidIndexer批摄取的。在`ioConfig`中`inputSpec`的`inputFormat`必须设置为 `"io.druid.data.input.avro.AvroValueInputFormat"`。
你可能想在`tuningConfig`中设置Avro读者模式`jobProperties`，如：`"avro.schema.path.input.value": "/path/to/your/schema.avsc"` or `"avro.schema.input.value": "your_schema_JSON_object"`,
如果读者模式没有设置，Avro对象包含文件模式将被使用，查阅 [Avro说明书](http://avro.apache.org/docs/1.7.7/spec.html#Schema+Resolution)。
确保包含"io.druid.extensions:druid-avro-extensions"作为扩展。


|字段|类型|描述|要求|
|-------|------|-------------|----------|
| type | String | This should say `avro_hadoop`. | no |
| parseSpec | JSON Object | Specifies the timestamp and dimensions of the data. Should be a timeAndDims parseSpec. | yes |
| fromPigAvroStorage | Boolean | Specifies whether the data file is stored using AvroStorage. | no(default == false) |

例如，使用Avro Hadoop解析自定义读者模式文件：
```json
{
  "type" : "index_hadoop",  
  "spec" : {
    "dataSchema" : {
      "dataSource" : "",
      "parser" : {
        "type" : "avro_hadoop",
        "parseSpec" : {
          "type": "timeAndDims",
          "timestampSpec": <standard timestampSpec>,
          "dimensionsSpec": <standard dimensionsSpec>
        }
      }
    },
    "ioConfig" : {
      "type" : "hadoop",
      "inputSpec" : {
        "type" : "static",
        "inputFormat": "io.druid.data.input.avro.AvroValueInputFormat",
        "paths" : ""
      }
    },
    "tuningConfig" : {
       "jobProperties" : {
          "avro.schema.path.input.value" : "/path/to/my/schema.avsc",
      }
    }
  }
}
```
