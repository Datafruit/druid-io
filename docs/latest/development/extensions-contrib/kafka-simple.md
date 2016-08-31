---
layout: doc_page
---

# Kafka简单的用户

## firehose

这是一个从kafka摄取数据的测试性firehose，使用kafka简单的用户api。
当前，这个firehose可以只在独立的实时节点工作。
KafkaSimpleConsumerFirehose的配置与kafka Eight Firehose相似，
除了`firehose`被`firehoseV2`代替如：

```json
"firehoseV2": {
  "type" : "kafka-0.8-v2",
  "brokerList" :  ["localhost:4443"],
  "queueBufferLength":10001,
  "resetOffsetToEarliest":"true",
  "partitionIdList" : ["0"],
  "clientId" : "localclient",
  "feed": "wikipedia"
}
```

|属性|描述|要求？|
|--------|-----------|---------|
|type|kafka-0.8-v2|yes|
|brokerList|list of the kafka brokers|yes|
|queueBufferLength|the buffer length for kafka message queue|no default(20000)|
|resetOffsetToEarliest|in case of kafkaOffsetOutOfRange error happens, consumer should starts from the earliest or latest message available|true|
|partitionIdList|list of kafka partition ids|yes|
|clientId|the clientId for kafka SimpleConsumer|yes|
|feed|kafka topic|yes|

为了能在生产中规模地使用消防带，建议设置至少三个复本，这意味着在`brokerList`至少有三个kafka代理。
为了让kafka话题每秒有1*10^4个事件,需保持一个分区可以正常工作,但如果需要更高的吞吐量可以添加多个分区。
