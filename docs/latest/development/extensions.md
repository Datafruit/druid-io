---
layout: doc_page
---

# Druid 扩展

Druid执行一个扩展系统,允许在运行时添加功能。扩展通常用于添加支持深存储(比如HDFS和S3),元数据存储(比如MySQL和PostgreSQL),新的聚合器,新的输入格式,等等。

生产集群将至少使用两个扩展；一个为深存储，另一个为元数据存储。很多集群也将使用额外的扩展。

## 包括扩展
 
请查阅[这里](../operations/including-extensions.html)。

## 核心扩展

核心扩展是Druid提交者维护的。

|名称|描述|文档|
|----|-----------|----|
|druid-avro-extensions|Support for data in Apache Avro data format.|[link](../development/extensions-core/avro.html)|
|druid-datasketches|Support for approximate counts and set operations with [DataSketches](http://datasketches.github.io/).|[link](../development/extensions-core/datasketches-aggregators.html)|
|druid-hdfs-storage|HDFS deep storage.|[link](../development/extensions-core/hdfs.html)|
|druid-histogram|Approximate histograms and quantiles aggregator.|[link](../development/extensions-core/approximate-histograms.html)|
|druid-kafka-eight|Kafka ingest firehose (high level consumer).|[link](../development/extensions-core/kafka-eight-firehose.html)|
|druid-kafka-extraction-namespace|Kafka-based namespaced lookup. Requires namespace lookup extension.|[link](../development/extensions-core/kafka-extraction-namespace.html)|
|druid-namespace-lookup|Required module for [lookups](../querying/lookups.html).|[link](../development/extensions-core/namespaced-lookup.html)|
|druid-s3-extensions|Interfacing with data in AWS S3, and using S3 as deep storage.|[link](../development/extensions-core/s3.html)|
|mysql-metadata-storage|MySQL metadata store.|[link](../development/extensions-core/mysql.html)|
|postgresql-metadata-storage|PostgreSQL metadata store.|[link](../development/extensions-core/postgresql.html)|

# 社区扩展
    
很多社区成员贡献了自己的扩展到Druid，不打包默认的Druid压缩文件。
社区扩展不是由Druid提交者维护的,尽管我们接受来自社区成员的补丁去使用这些扩展。
如果你想维护社区扩展，请发送请求到[druid-development group](https://groups.google.com/forum/#!forum/druid-development)让我们知道！

|名称|描述|文档|
|----|-----------|----|
|druid-azure-extensions|Microsoft Azure deep storage.|[link](../development/extensions-contrib/azure.html)|
|druid-cassandra-storage|Apache Cassandra deep storage.|[link](../development/extensions-contrib/cassandra.html)|
|druid-cloudfiles-extensions|Rackspace Cloudfiles deep storage and firehose.|[link](../development/extensions-contrib/cloudfiles.html)|
|druid-kafka-eight-simpleConsumer|Kafka ingest firehose (low level consumer).|[link](../development/extensions-contrib/kafka-simple.html)|
|druid-rabbitmq|RabbitMQ firehose.|[link](../development/extensions-contrib/rabbitmq.html)|
|graphite-emitter|Graphite metrics emitter|[link](../development/extensions-contrib/graphite.html)|

## 提升社区扩展为核心扩展
 
如果你想要提升扩展为核心，请[让我们知道](https://groups.google.com/forum/#!forum/druid-development)。
如果我们看到一个社区扩展被社区积极支持,我们可以基于社区的反馈提升它为核心。