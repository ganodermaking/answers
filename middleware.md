# 中间件
## rabbitmq
### rabbitMQ的工作原理？
##### 发送消息
* 生产者和Broker建立TCP连接。
* 生产者和Broker建立通道。
* 生产者通过通道消息发送给Broker，由Exchange将消息进行转发。
* Exchange将消息转发到指定的Queue（队列）

##### 接收消息
* 消费者和Broker建立TCP连接
* 消费者和Broker建立通道
* 消费者监听指定的Queue（队列）
* 当有消息到达Queue时Broker默认将消息推送给消费者。
* 消费者接收到消息。