# answers
Interview questions and answers

### redis哨兵主要功能？
* 集群监控：负责监控Redis master和slave进程是否正常工作；
* 消息通知：如果某个Redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员；
* 故障转移：如果master node挂掉了，会自动转移到slave node上；
* 配置中心：如果故障转移发生了，通知client客户端新的master地址。

### redis主从复制、哨兵和集群这三个有什么区别？
主从复制是为了数据备份，哨兵是为了高可用，redis主服务器挂了哨兵可以切换，集群则是因为单实例能力有限，搞多个分散压力。

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

#### TCP/UDP区别？
* TCP面向连接，UDP是无连接的。
* TCP提供可靠的服务，UDP尽最大努力交付，即不保证可靠交付。
* TCP面向字节流，UDP是面向报文的。
* TCP连接只能是点到点的，UDP支持一对一，一对多，多对一和多对多的交互通信。
* TCP首部开销20字节，UDP的首部开销小，只有8个字节。
* TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道。

### 两次握手为什么不可以？
采用三次握手是为了防止失效的连接请求报文段突然又传送到主机B，因而产生错误。

### 描述三次握手过程？
* 你招手
* 妹子点头微笑
* 妹子招手
* 你点头微笑
妹子先是点头微笑(回复对方)，然后再次招手(寻求确认)，将其合成一个动作，招手的同时点头微笑(syn + ack)。于是这四个动作就简化成了三个动作。

### 为什么要四次挥手？
TCP关闭链接四次握手原因在于tpc链接是全双工通道，需要双向关闭。client向server发送关闭请求，表示client不再发送数据，server响应。此时server端仍然可以向client发送数据，待server端发送数据结束后，就向client发送关闭请求，然后client确认。

### http与https有啥区别？https是怎么做到安全的？
https是通过对称加密、非对称加密和hash算法共同作用，来在性能和安全性上达到一个平衡。
通信的对象进行身份验证。