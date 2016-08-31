---
layout: doc_page
---
TopNMetricSpec
==================

topN指标规范指定应该如何排序topN值。
## Numeric TopNMetricSpec

最简单的指标规范是表明topN结果按指标排序的一个字符串值。它们都包括在topN查询：
```json
"metric": "<metric_name>"
```

指标段也可以由JSON对象给出。维度值按数值排序的语法如下：
```json
"metric": {
    "type": "numeric",
    "metric": "<metric_name>"
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|this indicates a numeric sort|yes|
|metric|the actual metric field in which results will be sorted by|yes|

## Lexicographic TopNMetricSpec

维度值按字母排序的语法如下：
```json
"metric": {
    "type": "lexicographic",
    "previousStop": "<previousStop_value>"
}
```

|property|description|required?|属性|描述|要求|
|--------|-----------|---------|
|type|this indicates a lexicographic sort|yes|
|previousStop|the starting point of the lexicographic sort. For example, if a previousStop value is 'b', all values before 'b' are discarded. This field can be used to paginate through all the dimension values.|no|

## AlphaNumeric TopNMetricSpec

维度值按字母数字顺序排序，即把不同于其他字符排序的值的数字进行排序。
该算法基于[https://github.com/amjjd/java-alphanum](https://github.com/amjjd/java-alphanum)。
```json
"metric": {
    "type": "alphaNumeric",
    "previousStop": "<previousStop_value>"
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|this indicates an alpha-numeric sort|yes|
|previousStop|the starting point of the alpha-numeric sort. For example, if a previousStop value is 'b', all values before 'b' are discarded. This field can be used to paginate through all the dimension values.|no|

## Inverted TopNMetricSpec

倒序排序维度值，即颠倒的顺序代表指标规范。这可以用升序来排序维度值。
```json
"metric": {
    "type": "inverted",
    "metric": <delegate_top_n_metric_spec>
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|this indicates an inverted sort|yes|
|metric|the delegate metric spec. |yes|
