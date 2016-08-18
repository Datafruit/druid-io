---
layout: doc_page
---

# 加载流

流可以摄入到Druid使用 [Tranquility](https://github.com/druid-io/tranquility) (Druid-aware 客户端)和[索引服务](../design/indexing-service.html)
或者通过独立的[实时节点](../design/realtime.html)。第一种方法的安装将更复杂，但是也提供了可伸缩性和高可用性的特点，可能需要先进的生产设置。
第二种方法可以了解其[局限性](../ingestion/stream-pull.html#limitations)。

## 流推

如果你有一个程序生成一个流，那么你可以实时直接把该流推进Druid。使用这个方法，Tranguility是嵌入在data-producing应用程序。
Tranguility绑定Storm和Samza处理器。这也可以从任何JVM-based程序获许直接API，如Spark 流或者Kafka消费者。

Tranguility为您无缝而且没有停息地处理分区，复制，服务搜索，和模式转换。你只需要定义你的Druid模式。

更多例子和信息，请查阅[Tranquility README](https://github.com/druid-io/tranquility)。

## 流拉

如果你想从外部服务拉数据，你有两个选择。最简单的选择是设置“复制”从数据源读取的服务和使用[流推方法](#stream-push)写到Druid。

另一选择是*流拉*。使用这个方法，Druid实时节点从[Firehose](../ingestion/firehose.html)摄取数据连接到你想要读的数据。
Druid为kafka,RabbitMQ,和各种其他的流系统包含内建的firehoses。

## 更多信息

对于更多通过基于推方法加载流数据的信息，请查阅 [这里](../ingestion/stream-push.html)。

更多通过基于拉方法加载流数据的信息，请查阅[这里](../ingestion/stream-pull.html)。