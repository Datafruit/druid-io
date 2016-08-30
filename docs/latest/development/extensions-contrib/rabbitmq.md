---
layout: doc_page
---

# RabbitMQ

## Firehose

#### RabbitMQFirehose

从定义消息队列中摄取firehose事件。
**注意：**增加**amqp-client-3.2.1.jar**到druid目录下使用这个Firehose。
消息队列firehose示例规范：
```json
"firehose" : {
   "type" : "rabbitmq",
   "connection" : {
     "host": "localhost",
     "port": "5672",
     "username": "test-dude",
     "password": "test-word",
     "virtualHost": "test-vhost",
     "uri": "amqp://mqserver:1234/vhost"
   },
   "config" : {
     "exchange": "test-exchange",
     "queue" : "druidtest",
     "routingKey": "#",
     "durable": "true",
     "exclusive": "false",
     "autoDelete": "false",
     "maxRetries": "10",
     "retryIntervalSeconds": "1",
     "maxDurationSeconds": "300"
   }
}
```

|属性|描述|默认|要求|
|--------|-----------|-------|---------|
|type|This should be "rabbitmq"|N/A|yes|
|host|The hostname of the RabbitMQ broker to connect to|localhost|no|
|port|The port number to connect to on the RabbitMQ broker|5672|no|
|username|The username to use to connect to RabbitMQ|guest|no|
|password|The password to use to connect to RabbitMQ|guest|no|
|virtualHost|The virtual host to connect to|/|no|
|uri|The URI string to use to connect to RabbitMQ| |no|
|exchange|The exchange to connect to| |yes|
|queue|The queue to connect to or create| |yes|
|routingKey|The routing key to use to bind the queue to the exchange| |yes|
|durable|Whether the queue should be durable|false|no|
|exclusive|Whether the queue should be exclusive|false|no|
|autoDelete|Whether the queue should auto-delete on disconnect|false|no|
|maxRetries|The max number of reconnection retry attempts| |yes|
|retryIntervalSeconds|The reconnection interval| |yes|
|maxDurationSeconds|The max duration of trying to reconnect| |yes|
