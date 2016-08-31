---
layout: doc_page
---

## Introduction 介绍

Druid可以使用Cassandra作为深存储机制。Segment和它们的元数据是存储在Cassandra的两个表中的：
`index_storage`和`descriptor_storage`。在Hood下面，Cassandra摄取影响Astyanax。索引存储表是一个[分块对象](https://github.com/Netflix/astyanax/wiki/Chunked-Object-Store)仓库。
它包含了分配给Historical节点的压缩Segment。由于segments可以扩大，分块对象存储允许集成多线程写到Cassandra，并将数据分散到集群中的所有节点。描述符存储表是一个正常的C*表存储segment metadatak。


## Schema
下面是创建语句：

```sql
CREATE TABLE index_storage(key text,
                           chunk text,
                           value blob,
                           PRIMARY KEY (key, chunk)) WITH COMPACT STORAGE;

CREATE TABLE descriptor_storage(key varchar,
                                lastModified timestamp,
                                descriptor varchar,
                                PRIMARY KEY (key)) WITH COMPACT STORAGE;
```

## 入门指南
首先创建schema。这一步我使用一个新的keySpace叫做`druid`，可以使用[Cassandra CQL `CREATE KEYSPACE`](http://www.datastax.com/documentation/cql/3.1/cql/cql_reference/create_keyspace_r.html) 命令创建。
然后，增加下面的语句到你的Historical和Realtime Runtime属性文件来启动Cassandra后台。
```properties
druid.extensions.loadList=["druid-cassandra-storage"]
druid.storage.type=c*
druid.storage.host=localhost:9160
druid.storage.keyspace=druid
```

如果你有问题，请用`druid-development@googlegroups.com`邮件发送过来，或者直接到`bone@alumni.brown.edu`
