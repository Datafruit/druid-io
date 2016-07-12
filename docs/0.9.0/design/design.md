---
layout: doc_page
---

为了全面了解Druid的框架，阅读[白皮书](http://static.druid.io/docs/druid.pdf)。请注意，Druid在快速发展，白皮书的内容可能过期。

什么是Druid
==============

Druid的创建允许获取大量很少变化的数据集，Druid设计的主要目的是作为一种服务面对代码部署、机器故障或者其他不可预测的事件发生的生产系统，能够保证100%的正常运行。

Druid现在可以做类似Dremel和PowerDrill的单表查询。同时增加了一些新特性：

1.  为局部嵌套数据结构提供列式存储格式
2.  分布式查询树模型
3.  为快速过滤做索引
4.  实时摄入数据和查询（摄入的数据是立即可用于查询）
5.  高容错的分布式体系架构

至于和其他系统的比较，Druid功能介于Dremel和PowerDrill之间，Druid实现了几乎Dremel的所有功能（Dremel可以处理任意嵌套数据结构，而Druid只能处理单一的数组类型），从PowerDrill借鉴了一下比较实用的数据布局和压缩方法。

Druid是一个大数据流、单一的数据摄取产品的不错的选择。特别是如果你的目标是想构建一个无停机和流入的数据以时间为导向的产品，Druid很适合。当谈查询速度的话题时，要明确一个很重要说明，"fast"的意义是什么：Druid完全有可能实现不到一秒钟完成对数以万亿行数据的查询（官方已经测试成功）。

### 总体架构

Druid的架构为不同作用的系统，这些系统结合起来就行成了一个可以工作的大系统。它的名字来源于很多类似于Druid的角色扮演游戏：它可以变换，能够通过不同的形式来实现不同角色。

每一个系统，或者说是组件，在下面有描述，更详细的信息在其专有的页面查看。你可以在右边的目录找到相应的页面，也可以通过点击下面相应组建的描述的链接跳转。

当前存在的节点类型：

* [**Historical**](../design/historical.html)  历史节点负责处理历史数据存储和查询历史数据（非实时），历史节点从“deep storage”下载segments，将结果数据返回给broker节点，historical加载完segment通知Zookeeper，Historical nodes使用Zookeeper监控需要加载或者删除哪些新的segments。
* [**Coordinator**](../design/coordinator.html) 协调节点对历史节点的分组进行监控，以确保数据可用，和最佳的配置。协调节点通过从元数据存储中读取元数据信息来判断哪些segments是应该加载到集群的，使用Zookeeper去判断哪些历史节点是存活的，在Zookeeper中创建任务条目告诉历史节点去加载和删除segments。
* [**Broker**](../design/broker.html) 代理节点接收来自外部client的查询请求，并转发这些请求给实时节点和历史节点，当代理节点接收到结果时，将来自实时节点和历史节点的结果合并返回给调用方。为了知道整个拓扑结构，代理节点通过使用Zookeeper在确定哪些实时节点和历史节点存活。
* [**Indexing Service**](../design/indexing-service.html) 索引服务节点由多个worker组成的集群，负责为加载批量的和实时的数据创建索引，并且允许对已经存在的数据进行修改。
* [**Realtime**](../design/realtime.html) 实时节点负责加载实时的数据到系统中，在生产使用的几个限制成本上实时节点比索引服务节点更容易搭建。

这种分离方式使每个节点只关心怎么才是使它运行的最好。通过把历史和实时处理分离开，我们把对内存的关注分离开，监听实时的流数据并处理它写入系统。我们分离协调节点和代理节点，我们把查询的需求分离开是为了在整个集群中维持良好的数据分布。

下图展示了通过这种架构如何查询数据和数据在这样的架构中是怎么传递的，以及在操作过程涉及到哪些节点。

<img src="../../img/druid-dataflow-3.png" width="800"/>

所有的节点都是以高可用的方式运行，无论是作为无共享集群或者热插拔故障转移节点都是同等的看待。

除了这些节点还有三个外部依赖系统：

1.  一个运行的 [ZooKeeper](../dependencies/zookeeper.html) 集群主要作用是帮助群集服务发现和维护当前数据的拓扑结构
2.  一个[metadata storage instance](../dependencies/metadata-storage.html) 元数据实例，用户维护系统中segments的元数据
3.  一个["deep storage" LOB store/file system](../dependencies/deep-storage.html) 深度存储系统，负责存储segments（如hdfs、S3）

下图展示了集群的管理层级，显示特定节点和依赖的外部系统是如何通过跟踪和交换元数据的方式管理集群的。

<img src="../../img/druid-manage-1.png" width="800"/>


### Segments and 数据存储

如上图所示，加载数据到Druid需要先进行索引。在创建索引的过程中Druid可以对数据进行分析、添加索引结构、压缩和调整布局尝试优化查询速度。对列数据进行一系列快速的操作列表如下：

-   转换为列模式
-   创建位图索引
-   使用各种压缩算法
    -   所有的列使用LZ4压缩
    -   所有的字符串列采用字典编码/标识以达到最小化存储
    -   对位图索引使用位图压缩

索引过程的输出成为一个"segment"（此处不想翻译成段，破坏了语意），Segments是Druid数据存储的基本存储结构。Segments包含一个数据集内的各种维度和度量值，以列的排列方式存储，以及这些列的索引。

Segments存储在LOB文件存储结构的"deep storage"中。在进行查询时，通过历史节点首先下载数据到历史节点的本地磁盘然后将磁盘数据映射到内存。

如果历史节点挂掉，它将不能对外提供查询它本地加载的Segments的服务，但是这些Segments还是存储在Deep Storage中，其他的历史节点还可以很轻松的把需要的Segments下载下来对外提供服务。这就意味着，即使从集群中移除所有的历史节点，也能够保证存储在Deep Storage中的Segments不会丢失。同时也意味着即使Deep Storage不可用，历史节点仍然可以对外服务，因为节点已经从Deep Storage中将相应的Segments拉取到了本地磁盘。

在集群中保存一个segment，需要将一条segment的元数据添加到元数据实例中，这条元数据是segment的自我描述，包含segment的schema、segment的大小、segment在Deep Storage中的存贮位置。这些元数据信息使Coordinator（协调器）知道集群能够对外提供哪些数据服务。

### 容错

-   **Historical** 正如上面所说的历史节点，即使一个历史节点挂掉，其他的历史节点也可以替代它，不必担心数据丢失。
-   **Coordinator** （协调器）可以配置成快速失败转移的方式运行，如果没有正常运行的协调器，对数据拓扑的改变请求将会停止（不会有新的数据进来，也不会对现有的数据做负载均衡），但是系统仍然能够继续运行。
-   **Broker**（代理节点）可以并行运行或者配置成快速失败转移的方式。
-   **Indexing Service** （索引服务）运行过程中对于摄入的任务进行备份，其中的coordination具有快速失败转移的功能。
-   **Realtime** （实时数据节点）依赖于传递数据流的语义，多个可以并行处理的相同的流，它们定期在磁盘中设置检查点，最终把这些检查点数据推送到Deep Storage。这样做是为了从失败中可以恢复数据。如果仅将数据保存在本地磁盘，可能会出现本地磁盘损坏不可用，就会造成磁盘数据丢失。
-   **"deep storage" 文件系统不可用，新的数据将不能够添加到集群中，但是集群仍然可以对外提供服务。
-   **metadata storage** 如果元数据存储不可用，Coordinator（协调器）将不能够从系统中发现新的segment，但Coordinator仍然可以通过现在已经存在的segment视图模式操作segment。
-   **ZooKeeper** 如果不可用，不能够对数据拓扑进行更改，但是代理节点仍然可以通过最近的数据拓扑视图对外提供查询服务。

### 查询处理

查询请求首先进入Broker（代理节点），代理节点将与已知存在的segment进行匹配查询。代理节点将选择一组机器，这组机器可以提供需要的segment的服务，将查询写入到这组机器的每台指定目标segment的服务器。在一次查询中，历史节点的查询和实时节点的查询的处理过程都会进行，然后返回结果。代理节点将历史节点和实时节点返回的结果合并，返回给查询请求方。甚至在看到一条数据之前，代理节点可以对不匹配的查询进行优化。

在代理节点优化数据的基础之上，可以采用更细粒度的索引。在看到结果数据之前，索引结构中的每一segment都允许历史节点来计算哪些行是符合过滤器要求的。过滤器可以对位图索引做所有的布尔判断计算，从来不会直接就看到数据。

一旦代理节点知道与当前查询相匹配的行，它就可以访问它所关心的列而不必再加载数据。将没用的查询数据扔掉。

### 内存使用

Druid不只是用内存，当我们第一次构建Druid时，Druid确实只是用内存，但是随着时间的推移，对成本和性能的权衡，会停止一直使用内存来操作数据，我们添加了对数据处理是用memory-map的能力，允许操作系统对内存中的数据进行分页处理。我们的生产集群首先要配置每一个提供服务节点的可用内存大小。

As you read some of the old blog posts or other literature about the project, you will see "in-memory" touted often, as that is the history of where Druid came from, but the technical reality is that there is a spectrum of price vs. performance. Being able to slide along that spectrum	 from all in-memory (high cost, great performance) to mostly on disk (low cost, low performance) is the important knob to be able to adjust.
当你读了一些旧的博客文章或其他有关项目的文献，你会经常看到对“in-memory”使用的鼓励，Druid历史节点的设计也来源于此，只不过技术的具体实现是性价比的综合考量。遵循这种考量作为重要的指标来调整内存（成本高，性能出色）和硬盘（成本低，性能低）存储的比例。
