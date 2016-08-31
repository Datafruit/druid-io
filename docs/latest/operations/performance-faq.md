---
layout: doc_page
---

# 性能常见的问题

## 我不能匹配你的基准测试结果

配置不当是迄今为止最大的问题,我们看到人们尝试部署Druid。教程中罗列的示例配置是专为单机上节点所在是一个小的数据。这个配置极少应用于生产中。

## 我应该怎么设置我的JVM堆？

JVM堆的大小取决于你运行Druid的类型节点。下面是一些注意事项。
[代理节点](../design/broker.html)使用JVM堆主要从历史和实时合并结果。代理也使用堆内存和处理线程groupBy查询。我们建议20G-30G的堆。

[Historical nodes](../design/historical.html)使用堆内存来存储中间结果,默认情况下,所有部分都是内存映射才可以查询。
通常,更多的内存可用历史的节点上,可以服务更多的部分没有数据分页到磁盘上的可能性。
在历史上,JVM堆用于[分组查询](../querying/groupbyquery.html),用于中间计算一些数据结构,和一般处理。
一种计算段有多少空间方法是： memory_for_segments = total_memory - heap - direct_memory - jvm_overhead。
注意total_memory这里指cgroup的可用内存(如果运行在Linux上),这为默认情况下是所有的系统内存。

我们建议为堆使用250mb*(processing.numThreads)。
[协调](../design/coordinator.html)不需要堆内存，堆用于加载所有段的信息来确定哪些段需要加载,下降,移动或复制。
## 中间计算缓冲区是什么？

中间计算缓冲区指定存储中间结果的缓冲区大小。历史和实时节点的计算引擎将使用一个这个尺寸的缓冲区做他们所有的中间计算堆。
更大的值允许更多聚合在一个经过数据而较小值可能需要更多的通过根据所执行的查询。默认大小是1073741824bytes(1GB)。

## 服务器最大的尺寸是多少？
服务器最大容量设置最大累积段大小(以字节为单位),可以容纳一个节点。通过在一个节点上控制内存/磁盘比率改变这个参数会影响性能。
设置该参数值大于总内存容量节点和可能导致磁盘上的分页。这个分页时间引入了一个查询延迟时间。

## 我的日志真的很健谈,我能让他们异步写？
是的,使用 `log4j2.xml` 类似于以下异步写一些更健谈的类:
```
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <AsyncLogger name="io.druid.curator.inventory.CuratorInventoryManager" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <AsyncLogger name="io.druid.client.BatchServerInventoryView" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <!-- Make extra sure nobody adds logs in a bad way that can hurt performance -->
    <AsyncLogger name="io.druid.client.ServerInventoryView" level="debug" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <AsyncLogger name ="com.metamx.http.client.pool.ChannelResourceFactory" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </AsyncLogger>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```
