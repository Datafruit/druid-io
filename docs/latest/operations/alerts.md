---
layout: doc_page
---
# Druid 警告

Druid在异常情况下生成警告。

警告在运行日志文件或通过HTTP(Apache Kafka等服务）时作为JSON对象发出。警告默认情况下是禁用的。

所有的Druid警告分享一个公共的字段组：

* `时间戳` - 创建警告的时间
* `服务` - 发出警告的服务名称
* `主机` - 发出警告的主机名称
* `严重性` - 警告的严重性如异常,出现组件故障,服务失败等等
* `描述` - 警告的描述
* `数据` - 如果有一个异常，那么JSON对象将有字段 `exceptionType`, `exceptionMessage` 和 `exceptionStackTrace`