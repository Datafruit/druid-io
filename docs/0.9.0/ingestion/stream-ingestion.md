---
layout: doc_page
---

# 读取流数据

流数据能够在Druid中利用[Tranquility](https://github.com/druid-io/tranquility)(一个Druid-aware客户端)和[indexing service](../design/indexing-service.html)，或者通过独立的[Realtime nodes](../design/realtime.html)接入。设置第一种方法会更复杂，但是提供了高级产品设置所需的扩展性和高可用性特性。第二种方法可以参考[limitations](../ingestion/stream-pull.html#limitations)。


## 推送流数据

如果一个项目产生了一个数据流，并实时地将这个数据流直接推送到Druid里。这个方法里，Tranquility是嵌入在data-producing应用里的。Tranquility绑定Storm and Samza 流处理器。同时提供API可以直接被任何基于JVM的项目调用，比如Spark流或者Kafka消费器。

Tranquility无缝地和不停机地处理分区，复制，服务发现和模式翻转。只需要定义Druid模式。

更多信息和实例，请参考[Tranquility README](https://github.com/druid-io/tranquility).

## 拉取数据流

如果想从一个外部服务拉取数据，有两种选择，最简单的是设置一个"copying"服务去读取数据源，然后用[stream push method](#stream-push)写入Druid。

另外一种是*stream pull*.用这种方法，Druid实时节点从连接想要读取的数据的[Firehose](../ingestion/firehose.html)接入数据。Druid为Kafka，RabbitMQ和各种其他的流数据系统提供内嵌的firehoses。

## 更多信息

更多基于推送方法读取流数据的信息，请参考[这里](../ingestion/stream-push.html)。
更多基于下拉方法读取流数据的信息，请参考[这里](../ingestion/stream-pull.html)。

