---
layout: doc_page
---

Peons
-----

查看peon的配置 [Peon Configuration](../configuration/indexing-service.html).

Peon在一个单独的jvm中运行任务，MiddleManager负责创建peons用于运行任务。Peons应该很少是单独启动运行（除非是为了测试）。

启动方式：
-------

Peon应该很少独立于middle manager之外运行，除非视为了开发的目的。

```
io.druid.cli.Main internal peon <task_file> <status_file>
```

task_file 包含任务信息的JSON对象。
status_file 指定任务状态输出到哪里。
