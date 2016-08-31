---
layout: doc_page
---

## 数据源

一个数据源相当于Druid数据库中的表。然而，一个查询也可以伪装成一个数据源,提供subquery-like功能。目前查询数据源支持只有[GroupBy](../querying/groupbyquery.html)查询。
### 表数据源

表数据源是最常用的一种类型。它可以通过一个字符串,或完整的结构来表示:

```json
{
	"type": "table",
	"name": "<string_value>"
}
```

### 数据源联合

数据源联合就是两个或者更多表数据源。
```json
{
       "type": "union",
       "dataSources": ["<string_value1>", "<string_value2>", "<string_value3>", ... ]
}
```


注意,数据源联合应该有相同的模式。
联合查询应该都是发送到代理/路由器节点和*不*支持直接通过历史节点。

### 查询数据源

查询数据源用于嵌套的groupBys而且现在只有groupBys支持。
```json
{
	"type": "query",
	"query": {
		"type": "groupBy",
		...
	}
}
```
