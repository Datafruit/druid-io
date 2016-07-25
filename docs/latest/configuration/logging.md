---
layout: doc_page
---
Logging 日志
==========================

Druid节点将发出用于调试控制台的日志。Druid节点也对自己状态发出周期性的指标。关于更多指标，查阅[配置](../configuration/index.html)。指标日志是默认打印到控制台的，可以用`-Ddruid.emitter.logging.logLevel=debug`禁用。
Druid使用日志[log4j2](http://logging.apache.org/log4j/2.x/)。日志可以用log4j2.xml文件配置。如果您想覆盖默认的Druid日志配置，请添加路径到您的类路径下包含log4j2.xml文件（例如_common / dir）的目录。注意这个目录应该比Druid库文件更早存在类路径中。最简单的方法是给类路径加前缀配置dir。为了使java日志通过log4j2，设置`-Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager`服务器参数。
log4j2.xml和Druid下config/_common/log4j2.xml的例子，一个示例如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{ISO8601} %p [%t] %c - %m%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>

    <!-- Uncomment to enable logging of all HTTP requests
    <Logger name="io.druid.jetty.RequestLog" additivity="false" level="DEBUG">
        <AppenderRef ref="Console"/>
    </Logger>
    -->
  </Loggers>
</Configuration>
```
