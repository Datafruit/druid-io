---
layout: doc_page
---
# Joins

Druid有限支持joins通过[query-time lookups](../querying/lookups.html)。
常见的query-time lookups用例是用另一个值(例如，一个人类可读的字符串值)替换一个维度的值(例如，一个字符串ID)。
这类似一个star-schema join。

Druid还没有完全支持joins。尽管Druid's的存储格式将允许joins的执行(包括没有失去列维度)，
完全支持连接尚未实现为以下原因:

1. 扩展连接查询，已经成为我们的专业经验里，一个常数与分布式数据库的瓶颈。
2. 增量收益功能被认为是管理价值低于预期的问题高并发，join-heavy工作负载。


连接查询是根据一组共享的密钥合并两个或多个数据流。主高级战略连接查询，我们意识到是hash-based策略或者sorted-merge策略。
hash-based策略要求所有一个数据集可以看起来像一个hash表，然后执行查找操作这对这个hash表“primary”流中的每一行。
sorted-merge策略假设每个流都是排序的连接键，从而允许增量加入的流。然而，每一种策略要么一些实体化的流顺序或hash表的格式。

当加入数量相当大的表(>10亿记录)，需要复杂的物化pre-join流分布式内存管理。
内存的复杂性管理是只有我们目标高度放大并发多租户工作负载的事实。