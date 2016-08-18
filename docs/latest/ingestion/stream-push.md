---
layout: doc_page
---

## Stream Push流推

Druid通过[Tranquility](https://github.com/druid-io/tranquility/blob/master/README.md)连接所有的流数据源，一个包实时推动流到Druid。Druid与Tranguility不绑定,你必须下载分布。

<div class="note info">
如果你从不加载流数据到Druid，我们建议先查看
<a href="../tutorials/tutorial-streams.html">流加载教程</a>然后再回到本页。
</div>

注意所有的流摄取选项，你必须确保引入的数据是最近的，足够（在当前时间[能够配置的窗口](#segmentgranularity-and-windowperiod)）。
旧信息将不会实时处理。历史的数据是最好的处理[批处理](../ingestion/batch-ingestion.html)。

### 服务器

Druid可以使用[Tranquility Server](https://github.com/druid-io/tranquility/blob/master/docs/server.md)，让你不需要开发一个JVM应用程序就可以发送数据到Druid。你可以运行Tranguility服务器用Druid中间管理者和历史进程托管。
Tranguility服务器通过下面代码启动：
```bash
bin/tranquility server -configFile <path_to_config_file>/server.json
```

自定义Tranguility服务器：
- 在`server.json`,自定义的`属性`和`数据源`。
- 如果你已经有服务器运行Tranguility，停止它们（CTRL-C）然后再重启。

自定义`server.json`的技巧，查阅*[Loading your own streams](../tutorials/tutorial-streams.html)*教程和[Tranquility 服务器文档](https://github.com/druid-io/tranquility/blob/master/docs/server.md)。

### Kafka
[Tranquility Kafka](https://github.com/druid-io/tranquility/blob/master/docs/kafka.md)
不需要写任何代码你便可以从Kafka加载数据到Druid。你只需要一个配置文件。
Tranquility服务器是这样启动的：
```bash
bin/tranquility kafka -configFile <path_to_config_file>/kafka.json
```

在单机自定义Tranguility Kafka快速入门配置：
- 在`kafka.json`,自定义`属性`和`数据源`。
- 如果你已经有服务器运行Tranguility，停止它们（CTRL-C）然后再重启。

自定义`kafka.json`的技巧，查阅[Tranquility Kafka 文档](https://github.com/druid-io/tranquility/blob/master/docs/kafka.md)。

### JVM apps and stream processorsJVM应用程序和流程序

Tranguility还可以嵌入到基于JVM的应用程序作为一个库。你可以直接在你自己的程序中使用[Core API](https://github.com/druid-io/tranquility/blob/master/docs/core.md),
或者您也可以使用连接器等流行的基于JVM的流处理器捆绑在Tranguility，如
[Storm](https://github.com/druid-io/tranquility/blob/master/docs/storm.md),
[Samza](https://github.com/druid-io/tranquility/blob/master/docs/samza.md),
[Spark Streaming](https://github.com/druid-io/tranquility/blob/master/docs/spark.md), 还有
[Flink](https://github.com/druid-io/tranquility/blob/master/docs/flink.md).
## 概念

### 任务创建

Tranguility自动创建Druid实时索引任务,处理分区,复制,服务发现和模式翻转,无缝而且没有停息时间。
你不需要直接编写代码来处理单个任务。但是,它可以帮助理解Tranguility创建任务。

Tranguility产生相对短暂的定期任务,每一个处理少量的(Druid段)(../design/segments.html)。Tranguility通过Zookeeper协调所有任务创建。
你可以尽可能多的Tranguility实例和相同的配置,甚至在不同的机器上,他们将发送到相同的任务组。
查阅 [Tranquility overview](https://github.com/druid-io/tranquility/blob/master/docs/overview.md)
了解更多关于Tranguility管理任务的细节。
### 段粒度和窗口期

段粒度是时间段覆盖的部分由每个任务产生。例如，“小时”段粒度的例子,一个段粒度将产生的任务创建段覆盖一个小时每一个。

窗口期是松弛时间允许事件。例如,一个窗口期十分钟(默认)意味着任何事件的时间戳大于开始时十分钟,或者比结束时多十分钟,将会被抛弃。

这些配置很重要,因为它们影响任务将活多长时间,以及数据在交给历史节点之前停留在实时系统多久。
例如,如果你的配置有段粒度“小时”和窗口期十分钟,任务将会继续监听事件一小时十分钟。出于这个原因,防止过度建设的任务,建议你窗口期小于段粒度。
### 追加

Druid流摄取是*追加*，意味着你在插入后不能使用流摄取更新或者删除个人的记录。如果你需要更新或者删除个人记录，你需要使用批处理重建索引过程。
查阅 *[批处理摄取](batch-ingestion.html)*页面了解更多细节。
Druid支持有效的删除整个时间范围而不用批量重建索引方法。
这个可以自动执行，通过设置保留政策。

### 保证

最优设计下Tranguility的运作。这个很难保存你的数据,通过允许您设置副本,由重试失败的推一段时间,但它并不能保证你的事件完全处理一次。在某些情况下,它可以减少或重复的事件:

- 你的配置窗口期外的时间戳事件将被删除。
- 如果你遭受比Druid中间管理者的失败比你配置副本统计,一些部分索引数据可能会丢失。
如果你遭受比配置副本更多的Druid中间管理者的失败，一些索引数据可能会丢失。
- 如果有一个持续的问题,防止通信与Druid索引服务,而且重试策略在那段时期是很麻烦的,或周期持续时间比窗口期长,一些事件将会被删除。
- 如果有问题,可以从索引服务收到的确认阻止Tranguility,它将重试批处理,从而导致重复的事件。
- 如果您使用的是 Storm或者Samza内部的Tranguility,各个部分的架构有一个至少一次设计和可能导致重复的事件。

在正常的操作下,这些风险是最小的。但是如果你需要绝对保真度为100%历史数据,我们建议使用[hybrid批处理/流](../tutorials/ingestion.html#hybrid-batch-streaming)体系结构。
## 文档

Tranguility文档可以在 [这里](https://github.com/druid-io/tranquility/blob/master/README.md)了解。
## 配置
Tranquility 配置可以在[here](https://github.com/druid-io/tranquility/blob/master/docs/configuration.md)了解。
Tranquility的优化配置可以在[这里](http://static.druid.io/tranquility/api/latest/#com.metamx.tranquility.druid.DruidTuning)了解。 
