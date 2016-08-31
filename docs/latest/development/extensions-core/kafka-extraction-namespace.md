---
layout: doc_page
---

# Kafka 命名空间查找

<div class="note caution">
查找是一个<a href="../experimental.html">试验</a>特性.
</div>

请确保 [包含](../../operations/including-extensions.html) `druid-namespace-lookup`和 `druid-kafka-extraction-namespace`作为一个扩展。
注意这个查找不会使用`pollPeriod`。
如果您需要尽可能迅速地更新填充,可以插入一个其键是旧值的kafka话题，而消息所需的是新值(也可以在utf-8)。

```json
{
  "type":"kafka",
  "namespace":"testTopic",
  "kafkaTopic":"testTopic"
}
```

|参数|描述|要求|默认|
|---------|-----------|--------|-------|
|`namespace`|The namespace to define|Yes||
|`kafkaTopic`|The kafka topic to read the data from|Yes||

## Kafka 重命名

`kafka-extraction-namespace` 扩展使得有name/key搭配的kafka阅读允许对维度值进行重命名。
例如去重命名一个ID为人们可以读懂的格式。

目前历史节点缓存的键/值对是从kafka通过MapDB进入一个短暂的内存映射。

## 配置

以下选项用于定义行为,应该包括扩展(所有查询服务节点)：

|属性|描述|默认|
|--------|-----------|-------|
|`druid.query.rename.kafka.properties`|A json map of kafka consumer properties. See below for special properties.|See below|

下面的针对kafka用户`druid.query.rename.kafka.properties`属性的处理

|属性|描述|默认|
|--------|-----------|-------|
|`zookeeper.connect`|Zookeeper connection string|`localhost:2181/kafka`|
|`group.id`|Group ID, auto-assigned for publish-subscribe model and cannot be overridden|`UUID.randomUUID().toString()`|
|`auto.offset.reset`|Setting to get the entire kafka rename stream. Cannot be overridden|`smallest`|

## 测试kafka重命名功能

为了测试这项设置，你可以通过以下生产控制台发送键/值配对到kafka流：

```
./bin/kafka-console-producer.sh --property parse.key=true --property key.separator="->" --broker-list localhost:9092 --topic testTopic
```

重命名可以在换行后（回车或者返回）发出`OLD_VAL->NEW_VAL`