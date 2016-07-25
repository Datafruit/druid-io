---
layout: doc_page
---

Druid vs Elasticsearch
======================

我们不是专家在搜索系统，如果我们的描述有不正确的地方，请通过邮件或者其他的方式给我们指出。
Elasticsearch是基于Apache Lucene的一个搜索系统。它提供了独立于文档的全文检索并提供原始事件水平数据的访问。
Elasticsearch增加更多的支持分析和聚合。[一些社区的成员](https://groups.google.com/forum/#!msg/druid-development/nlpwTHNclj8/sOuWlKOzPpYJ)有指出Elasticsearch的数据摄取和聚合资源需求远高于Druid。

Elasticsearch也不支持数据summarization/roll-up在摄入时，压缩数据需要真实的数据集存储到100x。这导致Elasticsearch拥有更大的存储需求。

Druid专注OLAP工作流。Druid是为了低成本的优化高性能（快速aggregation和ingestion），和支持广泛的分析操作。Druid有一些基本的搜索支持结构化事件数据，但不支持全文搜索。
Druid也不支持完全非结构化数据。Druid模式必须采取一些措施例如可以做到summarization/roll-up。