---
layout: doc_page
---

Druid vs SQL-on-Hadoop (Impala/Drill/Spark SQL/Presto)
===========================================================

SQL-on-Hadoop引擎为各种数据格式和数据存储提供一个执行引擎，而且很多可以叠加计算到Druid，同时提供一个SQL接口到Druid。


对于技术之间的直接比较，当只使用其中一个或者其他的，基本上可以归结为你产品需求和系统设计是什么。
Druid是为了

1. 一个总是在服务器上
1. 在real-time摄取数据
1. 处理slice-n-dice样式ad-hoc查询

SQL-on-Hadoop引擎一般回避Map/Reduce，反而直接从HDFS，或者有时候，从其他存储系统查询数据。
这些引擎（包括Impala和Presto）可以托管HDFS数据节点和配合他们实现查询数据本地化。
这意味着什么呢？我们可以从三个领域来谈
1. 查询
1. 数据摄取
1. 查询灵活性

### 查询

Druid Segment以自定义列格式存储数据。Segment可以直接扫描作为查询的一部分和每个Druid服务器计算最后合并在Broker层的一组结果。
这意味这在服务器之间转变的数据是查询和结果，而且所有的计算作为Druid服务器的一部分是内部地完成的。

大多数SQL-on-Hadoop引擎负责查询计划和执行底层存储层和存储格式。
即使没有查询运行(从Hadoop MapReduce消除JVM启动成本)这些过程还是会保存。
当数据在存储时，一些(Impala/Presto)SQL-on-Hadoop引擎的后台进程过程可以运行，几乎消除网络传输成本。
仍有一些延迟开销(例如serde time)从底层存储层获取数据到计算层。我们完全不知道这对性能有多少影响。
### 数据摄取

Druid允许实时摄取数据。你可以立即摄入和查询摄取数据，延迟事件反映在数据的速度是由需要多长时间交付事件Druid所决定的。

SQL-on-Hadoop，基于HDFS里的数据或其他支持存储，摄取率的数据是有限的摄取速率，支持存储数据。
一般来说，支持存储是最大的瓶颈对于数据速度变得可用。

### 查询灵活性

Druid的查询语言是相当低的水平和内部反应了Druid如何运作。
尽管Druid可以结合高级查询规划师[Plywood](https://github.com/implydata/plywood) 等支持大多数SQL查询和分析SQL查询(-大型表之间的连接)，
基础的Druid不如SQL-on-Hadoop通用的解决方案处理灵活。

SQL-on-Hadoop 支持完整连接的SQL样式查询。
## Druid vs Parquet
Parquet是一个列存储格式，是设计为SQL-on-Hadoop引擎工作的。Parquet没有查询执行引擎，而不是依靠外部资源中提取数据。

Druid的存储格式是为线性扫描高度优化。尽管Druid支持嵌套数据，Parquet的存储格式更多分层，为二进制分块设计的。理论上，这个应该在Druid上扫描更快。