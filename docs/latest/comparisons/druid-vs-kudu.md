---
layout: doc_page
---

Druid vs Kudu
=============

Kudu的存储格式能使单行升级，然而升级到现有的Druid Segment要求重创建Segment，因此，升级旧value的过程比Druid慢。
Kudu对于保持额外的头部空间存储更新的要求，和通过id而不是时间来组织数据具有潜力的介绍数据的一些额外的延时和访问一样，不需要在查询时解决一个查询。
Druid 在ingestion时summarizes/rollups数据，事实上减少行数据需要平均40倍的存储，而且需要大大地提高扫描行数据的性能。
然而，这个summarization过程丢失了关于个人事件的信息。Druid Segment也包含位图索引能快速过滤，这是Kudu现在不能支持的。Druid的Segment架构为了主要倾向于快速聚合和过滤，针对于OLAP工作流。
在Druid里增加是非常快的，然而更新老数据是比较慢的。这个是被设计为数据Druid是很好的典型的事件数据，而且不需要太频繁地更新。
Kudu支持唯一约束的任意主键，而且通过这些keys范围有效的查找。
Kudu选择不包括执行引擎，但是支持充分的操作只要允许执行引擎的node-local过程。
这意味着Kudu在相同的数据（如MR，Spark，和SQL）可以支持多框架。
