---
layout: doc_page
---
# 查询缓存


Druid支持通过一个LRU缓存将查询结果缓存。随着给定查询的参数，结果会存储在每个段的基础上。
Druid返回最终结果部分基于分段缓存和部分基于扫描的 historical/real-time segments结果。

Segment结果可以存储在一个本地堆缓存或分布式键/值存储在外部。段查询缓存可以在 Historicals和Broker级别上启用(不建议启用高速缓存)。
## 查询缓存代理


如果查询缓存启用 Historicals为小型集群那启用缓存代理可以产生更快的结果。这是因为建议设置较小的生产集群服务器(< 20 servers)。
注意,当缓存启用代理, Historicals的结果返回在每段的基础上，而且Historicals将无法做任何本地结果合并。

## Historicals 的查询缓存


较大的生产集群应该只有在Historicals为了避免使用Brokers合并所有查询结果时才启用高速缓存。
为使自己的本地结果合并和减少Brokers上的压力，应该启用 Historicals的缓存而不是启用Brokers