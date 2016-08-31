---
layout: doc_page
---

Querying 查询
===========

查询是由使用HTTP REST格式要求查询节点([Broker](../design/broker.html)，[Historical](../design/historical.html)，或者[Realtime](../design/realtime.html))。
查询是JSON上的
JSON查询和其每种节点类型表达相同的REST查询接口。
Druid原生查询语言是通过HTTP JSON，尽管许多社区成员贡献不同的[client libraries](../development/libraries.html)在其他语言查询Druid。
Druid原生查询是相对低水平的，紧密地映射到内部如何进行计算。Druid查询是轻量级的而且能非常快完成的。
这意味着对于更多分析，或者建立更毒复杂的可视化，可能需要多个Druid查询。

Available Queries
-----------------
Druid对于不同用例有多种查询类型。查询是由各种JSON属性和Druid对于不同用例有不同类型的查询组成。
各种查询类型的文档描述所有可以设置的JSON属性。
### 聚合查询

* [Timeseries](../querying/timeseriesquery.html)
* [TopN](../querying/topnquery.html)
* [GroupBy](../querying/groupbyquery.html)

### 元数据查询

* [Time Boundary](../querying/timeboundaryquery.html)
* [Segment Metadata](../querying/segmentmetadataquery.html)
* [Datasource Metadata](../querying/datasourcemetadataquery.html)

### 搜索查询

* [Search](../querying/searchquery.html)

我应该使用哪种查询？
-----------------------------------

如果可以，我们建议使用[Timeseries]()和[TopN]()查询而不是[GroupBy]()。GroupBy是最灵活的Druid查询，但是也有最差的性能。
Timeseries 比GroupBy查询快得多因为聚合不要求对维度分组。对于单维度分组于排序，topN查询比GroupBy更优化。

取消查询
----

可以用唯一的识别符明确地取消查询。如果查询识别符在查询时设置了，或否则，
以下端点可用于代理或路由器取消查询。
```sh
DELETE /druid/v2/{queryId}
```

例如，如果查询ID是`abc123`，查询可以被取消如下：
```sh
curl -X DELETE "http://host:port/druid/v2/abc123"
```
