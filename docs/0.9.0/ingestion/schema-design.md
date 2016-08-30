---
layout: doc_page
---

# 模式设计

这一页主要是为了帮助用户设计Druid待接入数据的模式。Druid摄入的非规范化的数据和列是三种类型之一：时间戳，维度，或者度量（或者Druid所提到的度量/聚合器）。在此之前，可以先了解一下OLAP数据的[标准命名约定](https://en.wikipedia.org/wiki/Online_analytical_processing#Overview_of_OLAP_systems)。

更多详细信息：

* Druid的每一行必须包括一个时间戳。数据总是按照时间来分区，每个查询都包含时间的过滤条件。查询结果同样能够根据时间段细分，比如分钟，小时，天等。
* 维度可以被过滤或者分组。维度可以是单字符串，也可以是字符串数组。
* 度量是指可以被聚合的字段。他们往往存储为数字（整数或浮点数），但也可以存储为复杂的对象，比如HyperLogLog或近似的直方图。


典型的生产表（或者Druid所提到的数据源）具有少于100个维度和少于100的度量，尽管有用户创建了上千个维度的数据源。

下面,我们概述一些模式设计的最佳做法：

## 高基数维度 (例如：唯一ID)

在实践中，我们往往不需要唯一ID的精确计数。存储唯一ID作为列将会结束[roll-up](../design/index.html)，并影响压缩。相反，存储唯一ID的数目的快照，并使用该快照作为聚合的一部分，将极大地提高性能（性能达到量级的提升），并且显著减少存储空间。Druid的hyperUnique聚合是基于关闭Hyperloglog的，可用于高基数维度的唯一计数。更多信息，请参考[这里](https://www.youtube.com/watch?v=Hpd3f_MLdXo)。

## 嵌套的维度

在写这篇文章的时候，Druid不支持嵌套的维度。嵌套维度需要被扁平化。例如，如果有以下形式的数据：
 
```
{"foo":{"bar": 3}}
```
 
然后索引它之前，应该把它转换为：

```
{"foo_bar": 3}
```

## 接入事件的计数

在摄入数据时，计数聚合器可用于计数摄取事件的数量。但是，需要注意的是，当查询这个指标时应该使用`longSum`聚合器。在查询时，一个` count`聚合器将返回Druid指定时间间隔内的行数，这可以用来确定roll-up比例是多少。

用一个例子来说明，如果ingestion spec包含：

```
...
"metricsSpec" : [
      {
        "type" : "count",
        "name" : "count"
      },
...
```

查询摄入的行数:

```
...
"aggregations": [
    { "type": "longSum", "name": "numIngestedEvents", "fieldName": "count" },
...
```

## 无模式维度

如果`dimensions`字段在ingestion spec中留空，Druid会处理非时间戳的列，被排除的维度或者作为维度的度量的每一列，应当注意，由于[#658](https://github.com/druid-io/druid/issues/658)， 这些segments将会比在字典顺序中被明确指定的维度列表稍大。此限制不影响查询的正确性-只是存储需求。

## 维度和度量中包含相同列

唯一ID的一个工作流程是能够过滤一个特定的ID，同时仍然能够做到ID列的快速唯一计数。如果没有使用无模式的维度，这个用例是通过设置不同的度量`name`和维度`name`来实现。如果正好使用的是无模式维度，在这里最好的做法是，使用相同的列两次，一次作为一个维度，另一次作为`hyperUnique`指标。这可能涉及到一些ETL的工作。

这是一个无模式维度的例子，使用了相同的列:

```
{"device_id_dim":123, "device_id_met":123}
```

`metricsSpec`包括:
 
```
{ "type" : "hyperUnique", "name" : "devices", "fieldName" : "device_id_met" }
```

`device_id_dim`应该被自动识别为维度。
