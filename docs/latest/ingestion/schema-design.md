---
layout: doc_page
---

# Schema设计

这个页面是用来帮助用户为Druid摄取数据设计一个schema。Druid摄取非规范的数据和列有三种类型：时间戳，维度，度量（在Druid里为度量/聚合）。
下面是[标准的命名约定](https://en.wikipedia.org/wiki/Online_analytical_processing#Overview_of_OLAP_systems) 


更多细节信息：

* Druid的每一行必须有个时间戳。数据以时间分区，而且每个查询有一个时间过滤器。查询结果还可分解为时间桶如分钟,小时,天,等等
* 维度是可以过滤或者分组的字段。它们要么是单一的字符串或者是一个字符串组。
* 度量是可以被聚合的字段。它们经常存储为数字（整型或者浮点型）但是也可以存储为复杂的对象如HyperLogLog草图或者大值的直方图草图。

典型的生产表（或者Druid里的数据源）有少于100个维度和少于100度量，尽管如此，基于用户的证词，数据源和无数的维度已经被创建。

下面。我们概述一些最好的schema设计实践：

## 高粒度维度（如唯一的IDs）

事实上，我们知道不需要那么精准的IDs数量。存储唯一的IDs为一个列将阻止[roll-up](../design/index.html)，而且影响其压缩。
反而，存储一个IDs数量的草图，和使用该草图作为聚合的一部分，将大大地提高性能（数量级的性能改进），而且很大意义地减少了存储。
Druid's `hyperUnique`聚合是基于HyperLogLog而且可以用于高粒度维度的独特计数。
更多信息，查阅 [这里](https://www.youtube.com/watch?v=Hpd3f_MLdXo).

## 嵌套维度

写这个的时候，Druid不支持嵌套维度。嵌套维度需要压缩。例如， 
 
```
{"foo":{"bar": 3}}
```
 
索引这个之前，你应该把它转换为： 
```
{"foo_bar": 3}
```

## 计算摄取事件的数量

在摄取时计数聚合器可以用来计数被摄取事件的数量。
而然，需要注意的是当你查询这个指标时，你应该使用一个`longSum`聚合器。一个`计数`聚合器在查询时将会每个时间间隔返回Druid行数，这个可以用来决定roll-up比例是多少。

使用一个例子解释，如果你的摄取规格包含：
```
...
"metricsSpec" : [
      {
        "type" : "count",
        "name" : "count"
      },
...
```

你应该使用以下方法查询摄取的行数： 
```
...
"aggregations": [
    { "type": "longSum", "name": "numIngestedEvents", "fieldName": "count" },
...
```

## Schema-less 维度

如果你的摄取规格里`维度`字段是空，Druid将会把不是时间戳的每个列，排除在外的维度，或者一个指标列作为维度。
这需要注意，因为 [#658](https://github.com/druid-io/druid/issues/658) 这些Segment将会略如果维度列表明确地指定在字典顺序中。
这个限制不会影响查询正确性-只是影响了存储需求。


## 包括相同的列作为维度和指标
                          
一个有唯一的IDs的工作流程是能够在特殊的ID中过滤，而且人能够成为最快的独特的ID列计数。
如果你不使用schema-less维度，这个用例支持通过设置指标的`name`与维度不同。
如果你正在使用schema-less维度，这里最好的用法是包含一些两个相同的列，一次作为维度，一次作为`hyperUnique`指标。这个可能涉及ETL时的一些工作。

如下示例，对于schema-less维度，重复相同的列：
```
{"device_id_dim":123, "device_id_met":123}
```

 而且在你的`指标规格`，包含：
```
{ "type" : "hyperUnique", "name" : "devices", "fieldName" : "device_id_met" }
```

`device_id_dim`应该作为维度自动获得。