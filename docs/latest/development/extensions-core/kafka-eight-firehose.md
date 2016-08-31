---
layout: doc_page
---

# Kafka Eight Firehose

请确保[包含](../../operations/including-extensions.html) `druid-kafka-eight`作为一个扩展。

这个消防带作为Kafka 0.8.x用户和从kafka摄取数据。
示例规范：

```json
"firehose": {
  "type": "kafka-0.8",
  "consumerProps": {
    "zookeeper.connect": "localhost:2181",
    "zookeeper.connection.timeout.ms" : "15000",
    "zookeeper.session.timeout.ms" : "15000",
    "zookeeper.sync.time.ms" : "5000",
    "group.id": "druid-example",
    "fetch.message.max.bytes" : "1048586",
    "auto.offset.reset": "largest",
    "auto.commit.enable": "false"
  },
  "feed": "wikipedia"
}
```

|属性|描述|要求|
|--------|-----------|---------|
|type|This should be "kafka-0.8"|yes|
|consumerProps|The full list of consumer configs can be [here](https://kafka.apache.org/08/configuration.html).|yes|
|feed|Kafka maintains feeds of messages in categories called topics. This is the topic name.|yes|
