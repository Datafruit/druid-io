---
layout: doc_page
---
# Refining Search Queries
search query spec定义一个搜索值和维度值之间怎样“匹配”。可用的search query spec有：
InsensitiveContainsSearchQuerySpec
----------------------------------

如果所有维度值包含指定在这个search query spec的值，忽视这种情况，会出现“匹配”。语法如下：
```json
{
  "type"  : "insensitive_contains",
  "value" : "some_value"
}
```

FragmentSearchQuerySpec
-----------------------

如果所有维度值包含所有指定在这个search query spec的值，默认这种情况，出现“匹配”。语法如下：
```json
{ 
  "type" : "fragment",
  "case_sensitive" : false,
  "values" : ["fragment1", "fragment2"]
}
```

ContainsSearchQuerySpec
----------------------------------

如果所有的维度值包含指定在这个search query spec的值，出现“匹配”。语法如下：
```json
{
  "type"  : "contains",
  "case_sensitive" : true,
  "value" : "some_value"
}
```

RegexSearchQuerySpec
----------------------------------

如果所有的维度值包含指定在这个search query spec的模式，出现“匹配”。语法如下：
```json
{
  "type"  : "regex",
  "pattern" : "some_pattern"
}
```