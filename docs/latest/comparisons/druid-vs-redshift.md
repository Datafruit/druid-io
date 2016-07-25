---
layout: doc_page
---
Druid vs Redshift
=================


### 怎样比较Druid和Redshift？

在画分区方面，Redshift以ParAccel(Actian)开始，Amazon允可有重大修改。
除了潜在的性能差异，还有一些功能的不同：

### Real-time数据摄取

因为Druid优化提供针对大量的流数据；这能够在real-time时加载和聚合数据。
一般地传统的数据仓库包括列存储，只有在批量摄取的时候才能工作而且不能对流数据定期地优化。

### Druid是读取对象分析的数据存储

Druid编写语义不是固定的也不支持完整的连接（我们支持大小表连接）。Redshift提供完整的SQL支持包括连接和插入/更新状态。
### 数据分配模型

Druid的数据分配是Segment-based和leverages一个高可用性“deep”存储如S3或者HDFS。按比例增加（或者减少）不需要大量的复制actions或者downtime；
事实上，丢失任何historical节点的数量不会导致数据丢失，因为新的historical节点总是可以从“deep”存储读取数据。

相反地，ParAccel的数据分配模型是hash-based。扩大集群要求re-hashing数据越过节点，这不用downtime很难执行。
Amazon’s Redshift解决这个问题的一个多步骤的过程:
* 设置集群为只读模式
* 从集群复制数据到已经存在于相同的新集群
* 流量重定向到新的集群

### Replication 复制策略

Druid使用segment-level数据分布意味着可以添加更多的节点和重新平衡，而不必执行交换。复制策略也使所有副本可供查询。复制是自动完成，而不影响性能。

ParAccel基于散列分布通常意味着通过热备用复制。你可以失去节点数量的数值限制而不丢失数据，而且复制策略通常不允许热备件帮助共享查询负载。  
### Indexing 策略

与列对象结构一样，当提供一个过滤器时Druid使用indexing结构加快查询执行速度。indexing结构增加存储费用（让它没那么容易突变化），但是也大大地加快了查询速度。
ParAccel没有使用indexing策略。