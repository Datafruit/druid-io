---
layout: doc_page
---
# ZooKeeper
Druid使用[ZooKeeper](http://zookeeper.apache.org/) （ZK）管理当前集群状态。Zk的操作是

1.  [Coordinator](../design/coordinator.html) 首选
2. Segment从 [Historical](../design/historical.html) 和 [Realtime](../design/realtime.html)“发布”协议
3. Segment加载/停止[Coordinator](../design/coordinator.html) 和 [Historical](../design/historical.html) 之间的协议
4.  [Overlord](../design/indexing-service.html)  首选
5.  [Indexing Service](../design/indexing-service.html) 任务管理


### Coordinator Leader Election

在路径方面我们使用Curator leadershipLatch 方法去做首选
```
${druid.zk.paths.coordinatorPath}/_COORDINATOR
```

### Segment从Historical和Realtime“发布”协议

`announcementsPath` 和 `servedSegmentsPath` 是为这个使用的。

所有的[Historical](../design/historical.html) 和 [Realtime](../design/realtime.html)节点在`announcementsPath`发布，特别地，它们会创建一个临时znode，在
```
${druid.zk.paths.announcementsPath}/${druid.host}
```

这表示它们是存在的。它们随后也将创建个永久的znode，在
```
${druid.zk.paths.servedSegmentsPath}/${druid.host}
```

当它们加载Segment时，会附加临时znode，如
```
${druid.zk.paths.servedSegmentsPath}/${druid.host}/_segment_identifier_
```

节点像[Coordinator](../design/coordinator.html) 和 [Broker](../design/broker.html)可以查看哪个节点是当前Segment。
### Segment加载/停止 Coordinator and Historical之间的协议

`loadQueuePath` 是为这个使用的。

当 [Coordinator](../design/coordinator.html) 决定 [Historical](../design/historical.html)节点应该加载或者停止一个Segment，写一个临时znode到
```
${druid.zk.paths.loadQueuePath}/_host_of_historical_node/_segment_identifier
```

这个节点将包含一个负载，表明Historical节点应该怎么处理给定的Segment。
当历史节点完成工作，为了表示协调完成，它将删除znode。