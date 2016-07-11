---
layout: doc_page
---

# 转换维值

以下的JSON文件可以用来查询维值操作。
## DimensionSpec

`DimensionSpec` 在聚合前定义维值如何被转换。
### 默认 DimensionSpec

返回维值和选择重命名维度。

```json
{ "type" : "default", "dimension" : <dimension>, "outputName": <output_name> }
```

### 提取DimensionSpec

用给定的[extraction function](#extraction-functions)返回维值
```json
{
  "type" : "extraction",
  "dimension" : <dimension>,
  "outputName" :  <output_name>,
  "extractionFn" : <extraction_function>
}
```

## Extraction 函数

提取函数定义每个维度的变换值。

转换可以运用于传统的（string）维，也可以作为特殊的`__time`维，根据查询代表当前time bucket [aggregation granularity](../querying/granularities.html).

**注意**: 对于函数用字符串值(比如regular expressions),
`__time` 维值在通过提取函数之前将会格式化为[ISO-8601 format](https://en.wikipedia.org/wiki/ISO_8601)

### Regular Expression Extraction 函数

返回第一个匹配的组给给定的regular expression。
如果没有匹配,它返回维度的值。

```json
{
  "type" : "regex", "expr" : <regular_expression>,
  "replaceMissingValue" : true,
  "replaceMissingValueWith" : "foobar"
}
```


例如，用`"expr" : "(\\w\\w\\w).*"` 将把这些`'Monday'`, `'Tuesday'`, `'Wednesday'` 转换成为 `'Mon'`, `'Tue'`, `'Wed'`.
    
如果 `replaceMissingValue`属性是真,提取函数变换维值不匹配正则表达式模式用户指定的字符串。默认值是 `false`。

`replaceMissingValueWith`属性设置字符串，如果`replaceMissingValue` 是true，不匹配的维值将会取代它。如果`replaceMissingValueWith` 是没有指明，不匹配的维值将会用空值替代。
例如，如果`expr` 是`"(a\w+)"`在上面的JSON例子中，regex匹配单词以字母`a`开始,提取函数将维度值如`banana`转换为 `foobar`。

### 部分 Extraction Function

如果regular expression匹配返回维值不变，否则返回null。
```json
{ "type" : "partial", "expr" : <regular_expression> }
```

### Search Query Extraction Function

如果给定[`SearchQuerySpec`](../querying/searchqueryspec.html)匹配返回维值不变，否则返回null。

```json
{ "type" : "searchQuery", "query" : <search_query_spec> }
```

### 提取子字符串函数


从提供的索引和所需的长度返回一个字符串的维度值。如果所需的长度超过维的长度值,字符串的其余部分将从索引返回。
如果索引大于维度的长度值,将返回null。
```json
{ "type" : "substring", "index" : 1, "length" : 4 }
```

子字符串的长度可以省略从索引返回剩余的维度值，或如果指数大于维度的长度值返回null。
```json
{ "type" : "substring", "index" : 3 }
```


### 时间格式提取函数

根据所给定的格式字符串、时区和语言环境返回维度值格式。

对于 `__time`维度值，通过[aggregation granularity](../querying/granularities.html)来区分这个时间值格式

对于一个常规的维度，它假设将字符串按[ISO-8601 date and time format](https://en.wikipedia.org/wiki/ISO_8601)格式

* `format` ：产生价值维度的日期时间格式，在[Joda Time DateTimeFormat](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html).
* `locale` :locale(语言和国家)使用,作为一个[IETF BCP 47 language tag](http://www.oracle.com/technetwork/java/javase/java8locales-2095355.html#util-text)，例如 `en-US`, `en-GB`, `fr-FR`, `fr-CA`，等。
* `timeZone` : 时区使用[IANA tz database format](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones)，例如 `Europe/Berlin` (这可能是聚合不同时区)

```json
{ "type" : "timeFormat",
  "format" : <output_format>,
  "timeZone" : <time_zone> (optional),
  "locale" : <locale> (optional) }
```

例如，下面的维度说明返回法国Montréal今天是本周中的哪一天：

```json
{
  "type" : "extraction",
  "dimension" : "__time",
  "outputName" :  "dayOfWeek",
  "extractionFn" : {
    "type" : "timeFormat",
    "format" : "EEEE",
    "timeZone" : "America/Montreal",
    "locale" : "fr"
  }
}
```

### Time Parsing Extraction Function

使用给定的输入格式解析维度值作为时间标记，并使用给定的输出格式返回格式化。

注意，如果你正在用`__time`维度，你应该考虑用[time extraction function instead](#time-format-extraction-function)替代，
因为工作时的值和字符串值是截然不同的。


时间格式的描述[SimpleDateFormat documentation](http://icu-project.org/apiref/icu4j/com/ibm/icu/text/SimpleDateFormat.html)
```json
{ "type" : "time",
  "timeFormat" : <input_format>,
  "resultFormat" : <output_format> }
```


### javascript提取函数

返回维度值，作为给定的javascript函数的转换

常规维度，是将输入值作为字符串传递。

 `__time` 维度，输入的值是通过一个数字来表示，也是UTC格式1970年1月1日以来的毫秒数

regular dimension示例
```json
{
  "type" : "javascript",
  "function" : "function(str) { return str.substr(0, 3); }"
}
```

```json
{
  "type" : "javascript",
  "function" : "function(str) { return str + '!!!'; }",
  "injective" : true
}
```

`injective`的属性指定如果javascript函数保持唯一性。默认值为 `false`意义的独特性不存在。

`__time`维度的示例:

```json
{
  "type" : "javascript",
  "function" : "function(t) { return 'Second ' + Math.floor((t % 60000) / 1000); }"
}
```

### Lookup extraction function 

Lookups是Druid在维度值(可选)替换为新的值上的一个概念。
使用查找更多的文档,请参阅[here](../querying/lookups.html)。
显式查询允许您指定一组键和值进行提取时使用。
```json
{
  "type":"lookup",
  "lookup":{
    "type":"map",
    "map":{"foo":"bar", "baz":"bat"}
  },
  "retainMissingValue":true,
  "injective":true
}
```

```json
{
  "type":"lookup",
  "lookup":{
    "type":"map",
    "map":{"foo":"bar", "baz":"bat"}
  },
  "retainMissingValue":false,
  "injective":false,
  "replaceMissingValueWith":"MISSING"
}
```

```json
{
  "type":"lookup",
  "lookup":{"type":"namespace","namespace":"some_lookup"},
  "replaceMissingValueWith":"Unknown",
  "injective":false
}
```

```json
{
  "type":"lookup",
  "lookup":{"type":"namespace","namespace":"some_lookup"},
  "retainMissingValue":true,
  "injective":false
}
```

lookup可以是`namespace` 或者`map`类型。`map` lookup是作为查询的一部分通过

`namespace`查找是根据处理查询填充在每个节点上[lookups](../querying/lookups.html)
`retainMissingValue`和`replaceMissingValueWith` 属性可以被指定在查询时提示如何处理缺失值。`replaceMissingValueWith`设置为`""`和设置`null`或省略的属性有同样的效果。

设置`retainMissingValue = true`是不合法的，还指定一个`replaceMissingValueWith`。

假设没有将多个组合成一个，`injective`的属性指定就可以使用优化。例如:如果ABC123是唯一的键映射到SomeCompany,那就可以优化,因为它是一个独特的查找。但如果ABC123和DEF456 SomeCompany地图,那不是一个独特的查找。将这个值设置为true和设置`retainMissingValue`为FALSE(默认)可能会导致不希望的行为。

属性`optimize`可以提供允许查询的优化提取过滤器(默认情况下`optimize = true`)。
优化层将在代理上运行,它将改写提取过滤器选择器过滤器的条款。
例如下面的filter

```json
{
    "filter": {
        "type": "extraction",
        "dimension": "product",
        "value": "bar_1",
        "extractionFn": {
            "type": "lookup",
            "optimize": true,
            "lookup": {
                "type": "map",
                "map": {
                    "product_1": "bar_1",
                    "product_3": "bar_1"
                }
            }
        }
    }
}
```

将会被重写成
```json
{
   "filter":{
      "type":"or",
      "fields":[
         {
            "filter":{
               "type":"selector",
               "dimension":"product",
               "value":"product_1"
            }
         },
         {
            "filter":{
               "type":"selector",
               "dimension":"product",
               "value":"product_3"
            }
         }
      ]
   }
}
```

A null dimension value can be mapped to a specific value by specifying the empty string as the key.
This allows distinguishing between a null dimension and a lookup resulting in a null.
零维值通过指定为空字符串作为键可以被映射到一个特定的值。
这可以区分零维度和null结果中的查找
例如，指定`{"":"bar","bat":"baz"}`和维值`[null, "foo", "bat"]`和替换缺失值`"oof"`将产生 `["bar", "oof", "baz"]`结果。
省略空字符串键将导致缺失值。例如,指定`{"bat":"baz"}` 与维值`[null, "foo", "bat"]`，用 `"oof"`代替缺失值将产生`["oof", "oof", "baz"]`结果 。
### Cascade Extraction 函数

提供链接的执行 extraction函数。
`extractionFns`的属性包括所有提取extraction函数的数组，是按照数组索引的顺序执行。
例子链接[regular expression extraction function](#regular-expression-extraction-function)， [javascript extraction function](#javascript-extraction-function)，和[substring extraction function](#substring-extraction-function)如下例子所示。

```json
{
  "type" : "cascade", 
  "extractionFns": [
    { 
      "type" : "regex", 
      "expr" : "/([^/]+)/", 
      "replaceMissingValue": false,
      "replaceMissingValueWith": null
    },
    { 
      "type" : "javascript", 
      "function" : "function(str) { return \"the \".concat(str) }" 
    },
    { 
      "type" : "substring", 
      "index" : 0, "length" : 7 
    }
  ]
}
```

它将按照指定extraction functions的名字顺序改变维值。
例如， `'/druid/prod/historical'`改变为 `'the dru'`过程是regular expression extraction function第一次转换它为 `'druid'`，然后javascript extraction function转换把它为 `'the druid'`，最后，substring extraction function转换将其为`'the dru'`。

根据给定的格式字符串返回dimension value formatted。

```json
{ "type" : "stringFormat", "format" : <sprintf_expression> }
```

例如如果你想合并"[" and "]"之前和之后的实际dimension value，您需要指定"[%s]" 格式字符串。
### Filtered DimensionSpecs

这些只是有效的multi-value dimensions。如果你在druid有一行，一个multi-value dimension与值["v1", "v2", "v3"]和根据[query filter](filters.html)值"v1"维度将你发送一个groupBy/topN查询分组。在响应中你会得到3行包含"v1", "v2"和"v3"。这种行为可能是直观的用例。

这是因为"query filter"是内部使用的 bitmaps,只用于匹配包含查询结果的处理的行。multi-value dimensions,"query filter"像一个包含检查，这将匹配dimension value["v1", "v2", "v3"]的行。请在 [segment](../design/segments.html)参阅"Multi-value columns"更多细节。

除了"query filter",能有效地选择要处理的行，你可以使用含有一个multi-value dimension值的filtered dimension spec过滤特定的值。
这些dimensionSpecs委托了DimensionSpec和一个过滤条件。从 "exploded"行，只有行匹配给定的过滤条件返回的查询结果。
以下filtered dimension spec充当白名单或黑名单过滤值按照"isWhitelist"属性值。
```json
{ "type" : "listFiltered", "delegate" : <dimensionSpec>, "values": <array of strings>, "isWhitelist": <optional attribute for true/false, default is true> }
```

filtered后dimension spec只保留匹配regex的值。请注意，用`listFiltered`来使用whitelist o或 blacklist usecase会比这个更快。

```json
{ "type" : "regexFiltered", "delegate" : <dimensionSpec>, "pattern": <java regex pattern> }
```

更多细节和示例，如 [multi-value dimensions](multi-value-dimensions.html)
### Upper and Lower extraction functions.

用大写或着小写返回dimension value。

可选的用户可以指定要使用的语言来执行大写或小写的转换
```json
{
  "type" : "upper",
  "locale":"fr"
}
```

或没有设置"locale"(在这种情况下,这个实例的默认语言环境的当前值是Java虚拟机)。
```json
{
  "type" : "lower"
}
```

### Lookup DimensionSpecs

<div class="note caution">
Lookups are an <a href="../development/experimental.html">experimental</a> feature.
</div>

Lookup DimensionSpecs可以直接用于定义一个查找实现dimension spec。
一般来说有两种不同类型的lookups implementations。
第一种是通过在查询的时用`map`实现。

```json
{ 
  "type":"lookup",
  "dimension":"dimensionName",
  "outputName":"dimensionOutputName",
  "replaceMissingValueWith":"missing_value",
  "retainMissingValue":false,
  "lookup":{"type": "map", "map":{"key":"value"}, "isOneToOne":false}
}
```

`retainMissingValue` 和 `replaceMissingValueWith`的属性可以在查询时提示如何处理缺失值。把`replaceMissingValueWith` 设置为 `""`和设置`null` 或省略属性具有同样的效果。
`retainMissingValue`设置为true，如果在查找中找不到，将会使用dimension's original value
默认值是`replaceMissingValueWith = null` 和 `retainMissingValue = false`这样会导致缺失值从而被视为失踪。
设置`retainMissingValue = true`和指定一个 `replaceMissingValueWith`是不合法的。
`injective` 属性指定如果没有将多个names组合成一个就可以使用优化假设.
`optimize`的属性允许基于extraction filter进行查询的优化(默认值为`optimize = true`)。
第二种是不可能通过在查询时由于其尺寸，将基于已经注册通过configuration file 或 coordinator的一个external lookup table或resource。
```json
{ 
  "type":"lookup"
  "dimension":"dimensionName"
  "outputName":"dimensionOutputName"
  "name":"lookupName"
}
```
