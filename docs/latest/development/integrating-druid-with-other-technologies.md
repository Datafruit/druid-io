---
layout: doc_page
---
# Druid与其他技术相结合
 
这个页面内容主要讨论我们怎样使用其他技术与Druid相结合。
## 使用开源流技术相结合
 
事件流可以存储在一个分布信息总线如kafka，而且可以通过分布流进程系统如Storm，Samza,或者Spark流进行深处理。
数据可以使用 [Tranquility](https://github.com/druid-io/tranquility)库通过流进程处理填充进Druid。

<img src="../../img/druid-production.png" width="800"/>

## 使用 SQL-on-Hadoop技术相结合

理论上Druid应该能和SQL-on-Hadoop技术相结合，如Apache Drill,Spark SQL,Presto,Impala,和Hive。