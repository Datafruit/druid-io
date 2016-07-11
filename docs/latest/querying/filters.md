---
layout: doc_page
---
#Query Filters
filter是一个JSON对象指示哪些行数据为查询时应该计算。它本质上是相当于在SQL WHERE子句。Druid支持以下类型的filters。
### Selector filter

最简单的filter是一个selector filter。selector filter将匹配一个特定的dimension与一个特定的值。对于更复杂的filters 的Boolean expressions，Selector filters可以用作 base filters。

SELECTOR filter 的语法如下：
``` json
"filter": { "type": "selector", "dimension": <dimension_string>, "value": <dimension_value_string> }
```


这相当于`WHERE <dimension_string> = '<dimension_value_string>'`
### Regular expression filter

regular expression filter类似于selector filter，但它是使用egular expressions。它与给定的模式匹配指定的dimension。这个模式可以用于任何标准[Java regular expression](http://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html)。
``` json
"filter": { "type": "regex", "dimension": <dimension_string>, "pattern": <pattern_string> }
```

### Logical expression filters

#### AND

AND filter的语法如下：
``` json
"filter": { "type": "and", "fields": [<filter>, <filter>, ...] }
```

fields中的filters 可以被这个页面的任何其他的filter定义。
#### OR

OR filter的语法如下：
``` json
"filter": { "type": "or", "fields": [<filter>, <filter>, ...] }
```


fields中的filters可以被这个页面的任何其他的filter定义。
#### NOT

NOT filter 的语法如下：
```json
"filter": { "type": "not", "field": <filter> }
```


指定的filter字段可以被这个页面的任何其他的filter定义。
### JavaScript filter

JavaScript filter对指定的JavaScript function匹配dimension。filter匹配值的函数返回true。
function接受一个参数，dimension value，并返回 true或false。
```json
{
  "type" : "javascript",
  "dimension" : <dimension_string>,
  "function" : "function(value) { <...> }"
}
```

**例子**
以下对dimension `name` 在`'bar'`和`'foo'`之间匹配任何dimension values
```json
{
  "type" : "javascript",
  "dimension" : "name",
  "function" : "function(x) { return(x >= 'bar' && x <= 'foo') }"
}
```

### Extraction filter

Extraction filter用一些指定[Extraction function](./dimensionspecs.html#extraction-functions)匹配一个dimension。
下面的filter匹配值为extraction function 转换为`input_key=output_value` 当`output_value`相当于filter的`value` 和`input_key`dimension。
**例子**
以下为column `product`匹配的dimension values`[product_1, product_3, product_5]`
```json
{
    "filter": {
        "type": "extraction",
        "dimension": "product",
        "value": "bar_1",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {
                    "product_1": "bar_1",
                    "product_5": "bar_1",
                    "product_3": "bar_1"
                }
            }
        }
    }
}
```
### Search filter

Search filters可以filter在部分将字符串匹配
```json
{
    "filter": {
        "type": "search",
        "dimension": "product",
        "query": {
          "type": "insensitive_contains",
          "value": "foo" 
        }        
    }
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This String should always be "search".|yes|
|dimension|The dimension to perform the search over.|yes|
|query|A JSON object for the type of search. See below for more information.|yes|

### In filter

In filter可以用来表达以下SQL查询:
```sql
 SELECT COUNT(*) AS 'Count' FROM `table` WHERE `outlaw` IN ('Good', 'Bad', 'Ugly')
```

IN filter的语法如下：
```json
{
    "type": "in",
    "dimension": "outlaw",
    "values": ["Good", "Bad", "Ugly"]
}
```

### Bound filter

Bound filter可以通过比较dimension values转成大写值或小写值来filter
使用默认的Comparison是基于字符串的而且**区分大小写的**。
你可以设置 `alphaNumeric`为`true`去使用numeric comparison
使用默认的bound filter不是一个strict inclusion`inputString <= upper && inputSting >= lower`
bound filter语法如下：
```json
{
    "type": "bound",
    "dimension": "age",
    "lower": "21",
    "upper": "31" ,
    "alphaNumeric": true
}
```

如果`21 <= age <= 31`保留列
```json
{
    "type": "bound",
    "dimension": "name",
    "lower": "foo",
    "upper": "hoo"
}
```

如果`foo <= name <= hoo`保留列
为了有strict inclusion用户可以设置`lowerStrict` 或`upperStrict`为`true`
有strict bounds：
```json
{
    "type": "bound",
    "dimension": "age",
    "lower": "21",
    "lowerStrict": true,
    "upper": "31" ,
    "upperStrict": true,
    "alphaNumeric": true
}
```

如果`21 <= age <= 31`保留列
有strict upper bound：
```json
{
    "type": "bound",
    "dimension": "age",
    "lower": "21",
    "upper": "31" ,
    "upperStrict": true,
    "alphaNumeric": true
}
```

如果`21 <= age <= 31`保留列
只有一个上限或者下限时
```json
{
    "type": "bound",
    "dimension": "age",
    "upper": "31" ,
    "upperStrict": true,
    "alphaNumeric": true
}
```

如果`age < 31`保留列
```json
{
    "type": "bound",
    "dimension": "age",
    "lower": "18" ,
    "alphaNumeric": true
}
```

如` 18 <= age`保留列
`alphaNumeric`的comparator，如果dimension value包括none-digits你可能期望得到 **fuzzy matching**。
如果dimension value以一个none digit开始，filter将考虑(`value < lowerBound` 和 `value > upperBound`)以外的范围。
如果dimension value以digit开始而且包含none digits 比较哪个更好。
如下限为`100`值是`10K`，filter将匹配(`100 < 10K` 返回 `true`)，因为`K`比任何的digit更好。
现在假设下限为`110`，filter将不会匹配(`110 < 10K` 返回 `false`)


#### Search Query Spec

##### Insensitive Contains

|属性|描述|要求|
|--------|-----------|---------|
|type|This String should always be "insensitive_contains".|yes|
|value|A String value to run the search over.|yes|

##### Fragment

|属性|描述|要求|
|--------|-----------|---------|
|type|This String should always be "fragment".|yes|
|values|A JSON array of String values to run the search over.|yes|
|caseSensitive|Whether strings should be compared as case sensitive or not. Default: false(insensitive)|no|

##### Contains

|属性|描述|要求|
|--------|-----------|---------|
|type|This String should always be "contains".|yes|
|value|A String value to run the search over.|yes|
|caseSensitive|Whether two string should be compared as case sensitive or not|yes|
