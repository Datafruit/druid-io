---
layout: doc_page
---

MiddleManager节点
------------------

Middle Manager的配置请查看 [Indexing Service Configuration](../configuration/indexing-service.html).

middle manager节点是执行提交任务的工作节点。middle manager将任务分发到peons运行，一个peon在一个单独的jvm中运行。
原因是我们通过单独的jvm对任务做资源隔离和日志隔离。一个peon在一个时间只能运行一个任务，然而，一个middle manager可以管理多个peon。

运行middle manager
-------

```
io.druid.cli.Main server middleManager
```

HTTP Endpoints
--------------

### GET

* `/status`

返回Druid的版本信息、加载的外部依赖、内存的使用、总共的内存和其他有关节点的信息。
