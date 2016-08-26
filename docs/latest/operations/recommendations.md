---
layout: doc_page
---

建议书
===================

# 使用UTC时区

我们建议你所有的事件和你的节点使用UTC时区,不仅是Druid,还有所有数据基础设施。这可以极大地减轻潜在的查询问题和不一致的时区。
查询non-UTC时区请查阅 [查询粒度](../querying/granularities.html#period-granularities)
# SSDs

历史和实时节点强烈推荐SSDs，如果你不完全在内存中运行一个集群。SSDs可以大大减少内存内外的记录数据所需的时间。
# 尽可能使用Timeseries和TopN查询而不是GroupBy
对于设计用例，Timeseries和TopN查询更优化并明显高于groupBy查询。从您的应用程序发布多个topN或timeseries查询可能会比一个groupBy查询更有效率。
# 段的大小问题

段一般应在300MB-700MB。许多小的段导致CPU利用率和效率低下，而太多的大的段影响查询性能,尤其是与TopN查询。

# Read FAQs阅读常见问题

你可以在这里阅读人们常见的问题：

1) [摄取常见问题](../ingestion/faq.html)

2) [性能常见问题](../operations/performance-faq.html)
