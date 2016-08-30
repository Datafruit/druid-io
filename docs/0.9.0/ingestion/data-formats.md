---
layout: doc_page
---
数据接入的格式
==========================

Druid可以接入非规范化的数据，例如JSON，CSV或分隔符形式如TSV，或任何自定义格式。而在这份文档中使用的大多数例子都是用JSON格式的数据文件，配置druid接入任何其他分隔符数据并不困难。我们欢迎贡献任何新的格式。

获取更多信息，请参考 [扩展列表](../development/extensions.html)。

## 格式化数据

下面是一些数据样例，来自于 [Wikipedia实例](../tutorials/quickstart.html)。

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

请注意，CSV和TSV数据不包含列头，这在指定要接入的数据时非常重要。

## 自定义格式

Druid 支持自定义的数据格式，并且能够使用正则表达式`Regex`解析器或者`JavaScript`解析器来解析相应格式。请注意，用任何一种解析器来解析数据的效率要低于本地java解析器或者外部流处理器。我们欢迎贡献新解析器。

## 配置

所有形式的Druid的接入需要某种形式的模式对象。数据格式是在`dataschema`下用`parsespec`指定的。

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

如果存在嵌套的JSON, [Druid能够自动地帮你展开](flatten-json.html).

### CSV

因为CSV文件不能包含列名（不允许带列头），在数据处理之前，必须添加：

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

`columns`字段的顺序必须与输入数据的列顺序一致。

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

`columns`字段的顺序必须与输入数据的列顺序一致。

确保将`delimiter`字段改为输入数据对应的分隔符。和CSV格式一样，必须指定列和需要被索引列的子集。

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

`columns`字段顺序必须与正则表达式匹配组的列顺序一致。如果列没有被提供，缺省列名（"column_1", "column2", ... "column_n"）将被分配。请确保列名包括所有的维度。

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

请注意，用JavaScript解析器时，数据必须完全被解析，并且在JS logic中以一个`{key:value}`格式返回。这意味着任何扁平化或者解析多维数值必须在这里处理。

### 多值维度

在TSV和CSV数据中，维度可以包括多个数值。可以设置`parseSpec`下`listDelimiter`来指定多值维度的分隔符。

JSON数据也可以包括多值维度。维度中的多个数值必须在接入数据中被格式化为一个JSON数组的形式。不需要额外配置`parseSpec`字段。
