---
layout: doc_page
---

# 元数据存储

元数据存储是Druid的外部依赖。Druid使用它来存储各种关于系统的元数据，而不是存储实际数据。
下面的表用于各种目的描述。

Derby是Druid默认的元数据存储，然而，这并不适合生产。
[MySQL](../development/extensions-core/mysql.html)和[PostgreSQL](../development/extensions-core/postgresql.html)是更多生产适用的元数据存储。
<div class="note caution">
Derby不适用于使用元数据存储生产。使用Mysql或者PostgreSQL代替。
</div>

## 使用derby

增加下面配置到你的Druid配置。
```properties
druid.metadata.storage.type=derby
druid.metadata.storage.connector.connectURI=jdbc:derby://localhost:1527//opt/var/druid_state/derby;create=true
```

## MySQL

查阅[mysql-metadata-storage 扩展](../development/extensions-core/mysql.html)
## PostgreSQL 

查阅[postgresql-metadata-storage](../development/extensions-core/postgresql.html). 

## 元数据存储表

### Segments 表

这个是通过 `druid.metadata.storage.tables.segments`属性决定的。

这个表存储关于系统中可用的Segment元数据。这个表由[Coordinator](../design/coordinator.html)调查确定Segment组，应该用于查询的系统。
表有两个主要功能列，其他列是索引目的。

`used`列是一个 boolean "tombstone"。1意味着Segment应该通过集群设置“used”（即它应该加载和用于请求）。
0意味着Segment不应该加载进集群。我们这样做没有真正地从集群移除片段消除他们的元数据（如果它是一个事件，这允许更简单的roll back。）

`payload`列存储JSON二进制，有所有的Segment元数据（一些数据存储在这个负载是冗余的，一些表中的列）。如以下：

```json
{
 "dataSource":"wikipedia",
 "interval":"2012-05-23T00:00:00.000Z/2012-05-24T00:00:00.000Z",
 "version":"2012-05-24T00:10:00.046Z",
 "loadSpec":{
    "type":"s3_zip",
    "bucket":"bucket_for_segment",
    "key":"path/to/segment/on/s3"
 },
 "dimensions":"comma-delimited-list-of-dimension-names",
 "metrics":"comma-delimited-list-of-metric-names",
 "shardSpec":{"type":"none"},
 "binaryVersion":9,
 "size":size_of_segment,
 "identifier":"wikipedia_2012-05-23T00:00:00.000Z_2012-05-24T00:00:00.000Z_2012-05-23T00:10:00.046Z"
}
```

注意，这个二进制的格式可以随时改变。
### Rule 表

rule表存储着不同的关于Segment存放位置的rules
这些rule通过[Coordinator](../design/coordinator.html)使用，当Segment（重新）分配决策的集群。
### 配置表

配置表存储Runtime配置对象。我们还没有很多这些表而且我们不确定是否将一直沿用这个机制，
但这是在Runtime时跨集群改变一些配置参数方法的开始。

### 相关任务表

在其工作进程中，也有大量的表被创建和被[Indexing Service](../design/indexing-service.html)使用。

### 审计表

审计表是用来存储配置变化的审计历史，如rule通过 [Coordinator](../design/coordinator.html)和其他配置变化。

## 访问：##

元数据存储只能通过以下访问：

1. Indexing Service 节点(如果有的话)
2. Realtime 节点(如果有的话)
3. Coordinator 节点

因此你需要给权限（如在AWS安全组）只对这些机器访问元数据存储。