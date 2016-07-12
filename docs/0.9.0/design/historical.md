---
layout: doc_page
---
Historical Node
===============
历史节点配置，请查看 [Historial Configuration](../configuration/historical.html).

历史节点负责加载历史Segment并且提供针对这些历史Segment的查询。
启动方式
-------

```
io.druid.cli.Main server historical
```

加载和Segment和提供对Segments的服务
----------------------------

每一个历史节点保持与Zookeeper的连接，查看一个可配置的关于新的Segment信息的Zookeepe路径。历史节点之间或者历史节点和协调节点之间不直接通信，而是通过Zookeeper管理协调。

[Coordinator](../design/coordinator.html) 协调节点负责将新的Segment分配给历史节点。分配是通过在Zookeeper下与历史节点相关联的加载队列路径下创建一个临时记录，想了解更多关于怎样配置协调节点分配Segment到历史节点，请参考协调节点的配置。

当一个历史节点发现在Zookeeper中与它关联的加载队列目录下有一个新的加载记录时，它首先检查本地磁盘目录（缓存）中关于新的Segment的信息。如果缓存中没有关于新的Segment的信息，历史节点将下载新的Segment的元数据信息并告知Zookeeper。元数据包含新的Segment在“Deep Storage”中的存储位置，怎样去解压缩和处理新的Segment的信息。对于更多关于Segment的元数据信息和Druid Segments请参考Segment的介绍。一旦一个历史节点处理完成一个Segment，该Segment在Zoookeeper与该节点关联的服务Segments路径中公布可以提供服务。此刻，这个Segment可以用于查询。

从在缓存中加载和服务Segments
---------------------------------------

回想一下，当一个历史节点注意到它的加载队列中一个新的Segment记录时，这个历史节点首先检查本地磁盘的缓存目录，查看新的Segment在之前是否下载过，如果本地缓存目录中已经存在该Segment的记录，历史节点将直接从本地磁盘读取Segment并且加载到内存中。

当历史节点第一次启动时，Segment的缓存就会被利用。在启动时，历史节点将从本地缓存目录中查找所有能被发现的Segments，直接加载并提供服务。这个特性使得只要历史节点在线就能很快提供查询服务。

查询Segments
-----------------

更多关于历史节点的查询信息，请查看 [Querying](../querying/querying.html)

一个历史可以配置为日志和报告的指标，为每一个查询服务。

HTTP 接口
--------------

历史节点可查询的的几个HTTP接口

### GET

* `/status`

返回Druid的版本信息，加载的外部依赖、内存的使用、总共的内存以及一些其他的有关历史节点的信息。

* `/druid/historical/v1/loadstatus`

返回一个标记说明是否所有本地缓存的segments都已经加载，这个标记值可以用于了解historical节点重启后，是否已经准备好提供查询服务。
