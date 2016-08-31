---
layout: doc_page
---
# Schema Changes

数据源的schemas任何时候都可以改变而且Druid支持不同schemas Segment。

## Replacing Segments

Druid使用数据源，区间，版本和分区号标识segments的唯一性。如果为一些时间粒度创建多个Segment，分区号只在Segment id可见。
例如，如果你有Segment，但是你一个小时内拥有的数据比单个Segment拥有的多，你可以为相同的时间创建多个Segment。
这些Segment将分享相同的数据源，区间和版本，但是有线程地增加分区号。

```
foo_2015-01-01/2015-01-02_v1_0
foo_2015-01-01/2015-01-02_v1_1
foo_2015-01-01/2015-01-02_v1_2
```

在上面Segment例子中，数据源=foo，区间=2015-01-01/2015-01-02，版本=v1，分区号=0。
如果将来你用新的schema重建索引数据，新创建的Segment将有更高版本的id。

```
foo_2015-01-01/2015-01-02_v2_0
foo_2015-01-01/2015-01-02_v2_1
foo_2015-01-01/2015-01-02_v2_2
```

Druid批处理索引（不是Hadoop-based也不是IndexTask-based）保证在区间基础上不分隔的更新。
例如，直到所有的`v2` segments 在Druid集群 `2015-01-01/2015-01-02`期间加载完，查询唯一地使用`v1`segments。
一旦所有的`v2`Segment能加载和查询，所有的查询忽视`v1`Segment和转换为`v2`segments。
然后，`v1`Segment是不能从集群被加载的。

注意，更新多个Segment区间只能在每个区间不可分隔。它们不会在更新的时候分开。
例如，你有Segment如下：

```
foo_2015-01-01/2015-01-02_v1_0
foo_2015-01-02/2015-01-03_v1_1
foo_2015-01-03/2015-01-04_v1_2
```

在Segment重叠的一段时间内，只要建立和替换 `v1` Segment，`v2`Segmens就将被加载到集群。
在v2段完全加载前，集群可能有 `v1` 和 `v2`Segment。
```
foo_2015-01-01/2015-01-02_v1_0
foo_2015-01-02/2015-01-03_v2_1
foo_2015-01-03/2015-01-04_v1_2
``` 
 
在这种情况下，查询可能点击`v1` 和 `v2`Segment。
## Segments中不同的Schemas

Druid Segment对于相同的数据源可能有不同的schemas。如果一个字符串列（维度）存在在一个Segment，但是别的不存在，两者Segment查询将继续工作。
查询对于Segment丢失维度将如维度没有值。
同样的，如果一个Segment有数值列（指标）但是其他的没有，Segment中查询丢失的指标将通常“做正确的事”。聚合这个丢失的指标行为就如指标丢失一样。