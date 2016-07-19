---
layout: doc_page
---
生产集群配置
================================

<div class="note info">
该配置是一个生产集群的一个例子。其他的硬件是可以组合的!便宜的硬件也是绝对可以的。
</div>
这个生产Druid集群假设元数据存储和Zookeeper已经建立。比如深存储是用[S3](https://aws.amazon.com/s3/)，[分布式缓存](http://memcached.org/)是用于分布缓存的。
<div class="note info">
这个例子中的节点不需要在他们自己个人的服务器上。Overlord 和 Coordinator节点应该集中在相同的硬件上。
</div>
响应（Historical，Broker和MiddleManager节点）查询的节点将尽可能地用核心，所以根据用法，把这些放在专用的机器上是最好的。有效地利用核的上限还没有什么特别的好处，而且它取决于查询类型，查询负载和模式。为正常使用，Historical后台进程应该至少每核心有1GB大小，但可以堆挤在一个更小的堆里测试。

因为内存中的缓存是良好性能必不可少的条件，而且越多的RAM会越好。Broker节点将用RAM缓存，所以他们做的不仅仅是路由查询。当他们都有比可用的内存更多段加载时，我们强烈推荐historical节点使用SSDs.
负责协调（Coordination和Overlord节点）的节点要求更少的处理。Zookeeper的核心利用效率，元数据存储，和Coordination节点像是每个过程/后台程序的1和2之间，所以他们可以多核心分享一个机器。这些后台程序工作的堆大小是在500MB和1GB之间。我们将用[EC2](https://aws.amazon.com/ec2/)r3.8xlarge节点给查询面对节点和m1.xlarge节点给coordination节点。
下面的例子相对较好地在生产中工作，然而，对于更优化调整我们选择的节点和更优化Druid集群的硬件两者都是可以的。
<div class="note caution">
对于高可用性，应该每个运行在单独的硬件上的过程至少有一个冗余副本。
</div>

### 通用配置（common.runtime.properties）

```
# Extensions
druid.extensions.loadList=["druid-s3-extensions", "druid-histogram", "mysql-metadata-storage"]

# Zookeeper
druid.zk.service.host=#{ZK_IPs}
druid.zk.paths.base=/druid/prod

druid.discovery.curator.path=/prod/discovery

# Request logging, monitoring, and metrics
druid.request.logging.type=emitter
druid.request.logging.feed=druid_requests

druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]

druid.emitter=http
druid.emitter.http.recipientBaseUrl=#{EMITTER_URL}

# Metadata storage
druid.metadata.storage.type=mysql
druid.metadata.storage.connector.connectURI=jdbc:mysql://#{MYSQL_URL}:3306/druid?characterEncoding=UTF-8
druid.metadata.storage.connector.user=#{MYSQL_USER}
druid.metadata.storage.connector.password=#{MYSQL_PW}

# Deep storage
druid.storage.type=s3
druid.s3.accessKey=#{S3_ACCESS_KEY}
druid.s3.secretKey=#{S3_SECRET_KEY}

# Caching
druid.cache.type=memcached
druid.cache.hosts=#{MEMCACHED_IPS}
druid.cache.expiration=2147483647
druid.cache.memcachedPrefix=d1
druid.cache.maxOperationQueueSize=1073741824
druid.cache.readBufferSize=10485760

# Indexing Service Service Discovery
druid.selectors.indexing.serviceName=druid:overlord

# Coordinator Service Discovery
druid.selectors.coordinator.serviceName=druid:prod:coordinator
```

### Overlord 节点

运行：

```
io.druid.cli.Main server overlord
```

硬件：

```
m1.xlarge (Cores: 4, Memory: 15.0 GB)
```
JVM配置

```
-server
-Xmx4g
-Xms4g
-XX:NewSize=256m
-XX:MaxNewSize=256m
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/overlord

# Only required if you are autoscaling middle managers
druid.indexer.autoscale.doAutoscale=true
druid.indexer.autoscale.strategy=ec2
druid.indexer.autoscale.workerIdleTimeout=PT90m
druid.indexer.autoscale.terminatePeriod=PT5M
druid.indexer.autoscale.workerVersion=#{WORKER_VERSION}

# Upload all task logs to deep storage
druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=druid
druid.indexer.logs.s3Prefix=prod/logs/v1

# Run in remote mode
druid.indexer.runner.type=remote
druid.indexer.runner.minWorkerVersion=#{WORKER_VERSION}

# Store all task state in the metadata storage
druid.indexer.storage.type=metadata
```

### MiddleManager 节点

运行：

```
io.druid.cli.Main server middleManager
```

硬件：

```
r3.8xlarge (Cores: 32, Memory: 244 GB, SSD)
```

JVM配置：

```
-server
-Xmx64m
-Xms64m
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/middlemanager

# Store task logs in deep storage
druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=#{LOGS_BUCKET}
druid.indexer.logs.s3Prefix=prod/logs/v1

# Resources for peons
druid.indexer.runner.javaOpts=-server -Xmx3g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
druid.indexer.task.baseTaskDir=/mnt/persistent/task/

# Peon properties
druid.indexer.fork.property.druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]
druid.indexer.fork.property.druid.processing.buffer.sizeBytes=536870912
druid.indexer.fork.property.druid.processing.numThreads=2
druid.indexer.fork.property.druid.segmentCache.locations=[{"path": "/mnt/persistent/zk_druid", "maxSize": 0}]
druid.indexer.fork.property.druid.server.http.numThreads=50
druid.indexer.fork.property.druid.storage.archiveBaseKey=prod
druid.indexer.fork.property.druid.storage.archiveBucket=aws-prod-druid-archive
druid.indexer.fork.property.druid.storage.baseKey=prod/v1
druid.indexer.fork.property.druid.storage.bucket=druid
druid.indexer.fork.property.druid.storage.type=s3

druid.worker.capacity=9
druid.worker.ip=#{IP_ADDR}
druid.worker.version=#{WORKER_VERSION}
```

### Coordinator 节点

运行：

```
io.druid.cli.Main server coordinator
```

硬件：

```
m1.xlarge (Cores: 4, Memory: 15.0 GB)
```
JVM配置：

```
-server
-Xmx10g
-Xms10g
-XX:NewSize=512m
-XX:MaxNewSize=512m
-XX:+UseG1GC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/coordinator
```

### Historical 节点

运行：

```
io.druid.cli.Main server historical
```

硬件：

```
r3.8xlarge (Cores: 32, Memory: 244 GB, SSD)
```

JVM配置：

```
-server
-Xmx12g
-Xms12g
-XX:NewSize=6g
-XX:MaxNewSize=6g
-XX:MaxDirectMemorySize=32g
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/historical

druid.historical.cache.useCache=true
druid.historical.cache.populateCache=true

druid.processing.buffer.sizeBytes=1073741824
druid.processing.numThreads=31

druid.server.http.numThreads=50
druid.server.maxSize=300000000000

druid.segmentCache.locations=[{"path": "/mnt/persistent/zk_druid", "maxSize": 300000000000}]

druid.monitoring.monitors=["io.druid.server.metrics.HistoricalMetricsMonitor", "com.metamx.metrics.JvmMonitor"]
```

### Broker节点

运行：

```
io.druid.cli.Main server broker
```

硬件：

```
r3.8xlarge (Cores: 32, Memory: 244 GB, SSD - this hardware is a bit overkill for the broker but we choose it for simplicity)
```

JVM配置：
```
-server
-Xmx25g
-Xms25g
-XX:NewSize=6g
-XX:MaxNewSize=6g
-XX:MaxDirectMemorySize=64g
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp

-Dcom.sun.management.jmxremote.port=17071
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/broker

druid.broker.http.numConnections=20
druid.broker.http.readTimeout=PT5M

druid.processing.buffer.sizeBytes=2147483647
druid.processing.numThreads=31

druid.server.http.numThreads=50
```

### Real-time 节点

运行：

```
io.druid.cli.Main server realtime
```

硬件（这是一个小过度）：
```
r3.8xlarge (Cores: 32, Memory: 244 GB, SSD - this hardware is way overkill for the real-time node but we choose it for simplicity)
```


JVM配置：
```
-server
-Xmx13g
-Xms13g
-XX:NewSize=2g
-XX:MaxNewSize=2g
-XX:MaxDirectMemorySize=9g
-XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+HeapDumpOnOutOfMemoryError

-Duser.timezone=UTC
-Dfile.encoding=UTF-8
-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
-Djava.io.tmpdir=/mnt/tmp

-Dcom.sun.management.jmxremote.port=17071
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

Runtime.properties:

```
druid.host=#{IP_ADDR}
druid.port=8080
druid.service=druid/realtime

druid.processing.buffer.sizeBytes=1073741824
druid.processing.numThreads=7

druid.server.http.numThreads=50

druid.monitoring.monitors=["io.druid.segment.realtime.RealtimeMetricsMonitor", "com.metamx.metrics.JvmMonitor"]
```
