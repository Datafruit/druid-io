---
layout: doc_page
---

# Graphite 发射器

## 介绍
 
这个扩展发出Druid指标到graphite carbon服务器。
[pickled](http://graphite.readthedocs.org/en/latest/feeding-carbon.html#the-pickle-protocol)后发送事件；
批处理的大小是可配置的。
## 配置

所有的graphite emitter配置参数是在 `druid.emitter.graphite`下面。

|属性|描述|要求|默认|
|--------|-----------|---------|-------|
|`druid.emitter.graphite.hostname`|The hostname of the graphite server.|yes|none|
|`druid.emitter.graphite.port`|The port of the graphite server.|yes|none|
|`druid.emitter.graphite.batchSize`|Number of events to send as one batch.|no|100|
|`druid.emitter.graphite.eventConverter`| Filter and converter of druid events to graphite event(please see next section). |yes|none|  
|`druid.emitter.graphite.flushPeriod` | Queue flushing period in milliseconds. |no|1 minute|
|`druid.emitter.graphite.maxQueueSize`| Maximum size of the queue used to buffer events. |no|`MAX_INT`|
|`druid.emitter.graphite.alertEmitters`| List of emitters where alerts will be forwarded to. |no| empty list (no forwarding)|
 
### Druid到Graphite的事件转换器 
 
Graphite事件转换器定义了一个druid指标名称扩展维度到一个Graphite指标路径之间的映射。
Graphite 指标路径是用以下模式组织的：

`<namespacePrefix>.[<druid service name>].[<druid hostname>].<druid metrics dimensions>.<druid metrics name>`
正确的命名指标是为了避免争议，混淆数据和潜在的错误解释。

`druid.historical.hist-host1_yahoo_com:8080.MyDataSourceName.GroupBy.query/time`示例：

 * `druid` -> 命名空间前缀
 * `historical` -> 服务器名称
 * `hist-host1.yahoo.com:8080` -> druid主机名称
 * `MyDataSourceName` -> 维度值
 * `GroupBy` -> 维度值
 * `query/time` -> 指标名称

我们有两种不同的事件转换执行方法：

#### 发送所有转换器
 
第一种方法叫做`all`,将发送所有的Druid服务器指标事件。

路径格式为`<namespacePrefix>.[<druid service name>].[<druid hostname>].<dimensions values ordered by dimension's name>.<metric>`
用户可以控制`<namespacePrefix>.[<druid service name>].[<druid hostname>].`
 
您可以通过设置 `ignoreHostname=true``druid.SERVICE_NAME.dataSourceName.queryType.query.time`删除主机名称。
您可以通过设置`ignoreServiceName=true``druid.HOSTNAME.dataSourceName.queryType.query.time`删除服务器名称。

```json

druid.emitter.graphite.eventConverter={"type":"all", "namespacePrefix": "druid.test", "ignoreHostname":true, "ignoreServiceName":true}

```

#### 基于white-list的转换器

第二种方法叫做`whiteList`,将只发送白名单列表的指标和维度。
与`all`转换器一样，用户可以控制`<namespacePrefix>.[<druid service name>].[<druid hostname>].`
基于white-list的转换器附带`./src/main/resources/defaultWhiteListMap.json`下的本地资源默认的白名单列表。

尽管用户可以通过应用`mapPath`属性重写默认的白名单列表。
这个属性是一个含有**白名单列表映射json对象**的文件路径的字符串。
例如下面的转换器将从文件`/pathPrefix/fileName.json`读取地址。

```json

druid.emitter.graphite.eventConverter={"type":"whiteList", "namespacePrefix": "druid.test", "ignoreHostname":true, "ignoreServiceName":true, "mapPath":"/pathPrefix/fileName.json"}

```

**当Druid发出大量的指标时，我们强烈建议使用`whiteList`转换器**