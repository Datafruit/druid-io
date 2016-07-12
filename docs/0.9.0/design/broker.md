---
layout: doc_page
---
Broker
======
关于broker的更多配置信息, 请参考 [Broker Configuration](../configuration/broker.html).

在Druid分布式集群，broker（代理）提供针对segment的路由查询。代理节点从Zookeeper获取Segments存储在哪些节点和怎样找到正确的节点的元数据。代理节点还可以把来自于所有单个节点的结果合并在一起。在启动时，实时节点在Zookeeper中声明他们自己和他们服务的Segments。
Running
-------

```
io.druid.cli.Main server broker
```

转发查询
------------------

大多数Druid的查询包含一个区间对象，表明在一个时间段内的数据请求。同时，Druid Segments划分成一些包含时间段的数据，Segments分布在集群中。考虑到一个简单的数据源包含7个Segment，每一个Segment包含一个周中一天的数据。对数据源发出的任何查询如果时间段超过一天将会匹配多个Segment。这些Segments很可能分布在多个节点，因此，这种查询很可能从多个节点中检索数据。

确定哪些节点转发查询，代理节点首先在Zookeeper中建立一种全局视图。Zookeeper维护有关历史和实时的节点信息和他们所能提供服务的Segment。在Zookeeper的每一个数据源，代理节点建立相关Segments的时间轴和为这些Segments提供服务的节点。当收到一个特定数据源和时间间隔的查询请求，代理节点执行查找与查询数据源时间间隔相关的时间轴和检索包含数据查询的节点。代理节点然后将查询转发到所选节点。

缓存
-------

代理节点缓存采用LRU缓存失效策略。代理节点缓存存储已经查询过的每个Segment结果，缓存可以在每个代理节点的本地存储，也可以使用外部跨多个节点的分布式缓存，如Memcached。每次一个代理节点接收到一个查询请求，首先将这个查询映射到一组Segments，这些Segment结果的子集可能在缓存中已经存在，在缓存中已经存在的结果可以被直接拉取。对于一些缓存中不存在的结果。代理节点会转发查询到历史节点。一旦历史节点返回其结果，代理节点将结果存储到缓存中。实时节点返回的结果将永远不会被缓存，因此实时节点的查询请求将永远被转发到实时节点。实时节点的数据是不断变化的，缓存实时节点的结果是不可靠的。

HTTP交互
--------------

代理节点提供几个HTTP交互命令

### GET

* `/status`

返回Druid的版本，加载扩展，内存使用，总内存和其他关于节点有用的信息。

* `/druid/v2/datasources`

返回一个可查询的数据源列表

* `/druid/v2/datasources/{dataSourceName}`

返回数据源的维度和度量。可以提供一个参数“full”获取这些时间间隔内提供服务的维度和度量的时间间隔列表。也可以提供一个参数“interval”明确指定一个特定的时间间隔。
如果没有指定时间间隔，默认时间间隔是在当前时间将被使用之前生成一个可配置的时间段。这个时间间隔被指定使用ISO8601格式：
druid.query.segmentMetadata.defaultHistory
* `/druid/v2/datasources/{dataSourceName}/dimensions`

返回数据源的维度

* `/druid/v2/datasources/{dataSourceName}/metrics`

返回数据源度量指标

* `/druid/broker/v1/loadstatus`

如果代理节点知道在Zookeeper中的所有Segment，将返回一个标志。这可以用来标记当代理节点重启之后已经准备好可以接收查询请求。
