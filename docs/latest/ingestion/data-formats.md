---
layout: doc_page
---
摄取的数据格式
==========================

Druid可以摄取JSON，CSV，或者如ISV的限定格式，或者任何自定义格式的非规范化数据。文档中大多数例子是使用JSON格式的数据，配置Druid摄取其他的限定数据并不难。
我们欢迎贡献任何新的格式。

对于额外的数据格式，请查阅我们 [扩展列表](../development/extensions.html)。
## 格式化的数据

下面是一些使用在[Wikipedia example](../tutorials/quickstart.html)上数据的示例。
_JSON_

```json
{"timestamp": "2013-08-31T01:02:33Z", "page": "Gypsy Danger", "language" : "en", "user" : "nuclear", "unpatrolled" : "true", "newPage" : "true", "robot": "false", "anonymous": "false", "namespace":"article", "continent":"North America", "country":"United States", "region":"Bay Area", "city":"San Francisco", "added": 57, "deleted": 200, "delta": -143}
{"timestamp": "2013-08-31T03:32:45Z", "page": "Striker Eureka", "language" : "en", "user" : "speed", "unpatrolled" : "false", "newPage" : "true", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Australia", "country":"Australia", "region":"Cantebury", "city":"Syndey", "added": 459, "deleted": 129, "delta": 330}
{"timestamp": "2013-08-31T07:11:21Z", "page": "Cherno Alpha", "language" : "ru", "user" : "masterYi", "unpatrolled" : "false", "newPage" : "true", "robot": "true", "anonymous": "false", "namespace":"article", "continent":"Asia", "country":"Russia", "region":"Oblast", "city":"Moscow", "added": 123, "deleted": 12, "delta": 111}
{"timestamp": "2013-08-31T11:58:39Z", "page": "Crimson Typhoon", "language" : "zh", "user" : "triplets", "unpatrolled" : "true", "newPage" : "false", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Asia", "country":"China", "region":"Shanxi", "city":"Taiyuan", "added": 905, "deleted": 5, "delta": 900}
{"timestamp": "2013-08-31T12:41:27Z", "page": "Coyote Tango", "language" : "ja", "user" : "cancer", "unpatrolled" : "true", "newPage" : "false", "robot": "true", "anonymous": "false", "namespace":"wikipedia", "continent":"Asia", "country":"Japan", "region":"Kanto", "city":"Tokyo", "added": 1, "deleted": 10, "delta": -9}
```

_CSV_

```
2013-08-31T01:02:33Z,"Gypsy Danger","en","nuclear","true","true","false","false","article","North America","United States","Bay Area","San Francisco",57,200,-143
2013-08-31T03:32:45Z,"Striker Eureka","en","speed","false","true","true","false","wikipedia","Australia","Australia","Cantebury","Syndey",459,129,330
2013-08-31T07:11:21Z,"Cherno Alpha","ru","masterYi","false","true","true","false","article","Asia","Russia","Oblast","Moscow",123,12,111
2013-08-31T11:58:39Z,"Crimson Typhoon","zh","triplets","true","false","true","false","wikipedia","Asia","China","Shanxi","Taiyuan",905,5,900
2013-08-31T12:41:27Z,"Coyote Tango","ja","cancer","true","false","true","false","wikipedia","Asia","Japan","Kanto","Tokyo",1,10,-9
```

_TSV (Delimited)_

```
2013-08-31T01:02:33Z	"Gypsy Danger"	"en"	"nuclear"	"true"	"true"	"false"	"false"	"article"	"North America"	"United States"	"Bay Area"	"San Francisco"	57	200	-143
2013-08-31T03:32:45Z	"Striker Eureka"	"en"	"speed"	"false"	"true"	"true"	"false"	"wikipedia"	"Australia"	"Australia"	"Cantebury"	"Syndey"	459	129	330
2013-08-31T07:11:21Z	"Cherno Alpha"	"ru"	"masterYi"	"false"	"true"	"true"	"false"	"article"	"Asia"	"Russia"	"Oblast"	"Moscow"	123	12	111
2013-08-31T11:58:39Z	"Crimson Typhoon"	"zh"	"triplets"	"true"	"false"	"true"	"false"	"wikipedia"	"Asia"	"China"	"Shanxi"	"Taiyuan"	905	5	900
2013-08-31T12:41:27Z	"Coyote Tango"	"ja"	"cancer"	"true"	"false"	"true"	"false"	"wikipedia"	"Asia"	"Japan"	"Kanto"	"Tokyo"	1	10	-9
```

注意 CSV 和 TSV数据没有包含列标题。当你指定摄取数据时这个是很重要的。
## 自定义格式
Druid支持自定义数据格式和可以使用`Regex` 解析器或者 `JavaScript`解析器去解析这些格式。请注意，使用这两种解析器解析数据，将不会和本地Java解析或者使用外部的流处理器有一样的效果。
我们欢迎贡献新的解析器。

## 配置

所有Druid摄取格式要求一些schema对象格式。摄取数据的格式是用 `dataSchema`的`parseSpec`入口指定。

### JSON

```json
  "parseSpec":{
    "format" : "json",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "dimensionSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

如果你有嵌套的JSON，[Druid 可以自动为你压缩](flatten-json.html)。
### CSV

因为CSV数据不能包含列名称（没有header），必须先增加这些数据才能处理：

```json
  "parseSpec":{
    "format" : "csv",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "columns" : ["timestamp","page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city","added","deleted","delta"],
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

 `columns`字段必须匹配你输入数据相同顺序的column。
### TSV

```json
  "parseSpec":{
    "format" : "tsv",
    "timestampSpec" : {
      "column" : "timestamp"
    },
    "columns" : ["timestamp","page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city","added","deleted","delta"],
    "delimiter":"|",
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    }
  }
```

`columns`字段必须匹配你输入数据相同顺序的column。

一定要改变 `分隔符`去适当的分隔你的数据。如CSV，你必须指定列和你想要索引的列的子集。
### Regex

```json
  "parseSpec":{
    "format" : "regex",
    "timestampSpec" : {
      "column" : "timestamp"
    },        
    "dimensionsSpec" : {
      "dimensions" : [<your_list_of_dimensions>]
    },
    "columns" : [<your_columns_here>],
    "pattern" : <regex pattern for partitioning data>
  }
```

`columns`字段匹配同样顺序的Regex匹配组的列。
如果没有提供列，默认列名("column_1", "column2", ... "column_n")将会代替。确保你的列名包含你所有的维度。

### JavaScript

```json
  "parseSpec":{
    "format" : "javascript",
    "timestampSpec" : {
      "column" : "timestamp"
    },        
    "dimensionsSpec" : {
      "dimensions" : ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"]
    },
    "function" : "function(str) { var parts = str.split(\"-\"); return { one: parts[0], two: parts[1] } }"
  }
```

请注意用JavaScript解析器的数据必须充分解析并返回JS逻辑格式`{key:value}`。
这意味着所有的压缩或者解压多维的值必须在这里完成。

### 多值维度

维度可以有多个ISV和CSV数据值。为多值维度指定分隔符，在 `parseSpec`设置 `listDelimiter`
JSON数据也可以包含多值维度。维度的多个值必须被格式化为摄入数据的JSON数组。不需要额外的`parseSpec`配置。