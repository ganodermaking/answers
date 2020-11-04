# 数据库
<!-- GFM-TOC -->
* mysql
  * [Mysql索引原理](#Mysql索引原理)
  * [Mysql事务的基本要素（ACID）](#Mysql事务的基本要素（ACID）)
  * [Mysql事务的并发](#Mysql事务的并发)
  * [Mysql分布式事务](#Mysql分布式事务)
  * [Mysql Having和Where的区别](#Mysql Having和Where的区别)
  * [Mysql浮点数](#Mysql浮点数)
  * [Mysql日志系统](#Mysql日志系统)
* redis
  * [Redis缓存问题](#Redis缓存问题)
  * [Redis哨兵模式](#Redis哨兵模式)
  * [Redis集群模式](#Redis集群模式)
  * [单线程的Redis为什么这么快](#单线程的Redis为什么这么快)
  * [Redis的过期策略](#Redis的过期策略)
<!-- GFM-TOC -->

## mysql
### Mysql索引原理
Mysql Innodb支持事务、外键、行级锁

#### 为什么用B+树实现
* B+树能显著减少IO次数，提高效率；
* B+树查询效率更加稳定，因为数据放在叶子节点；
* B+树能提高范围查询的效率，因为叶子节点指向下一个叶子节点。

#### MyISAM与InnoDB索引比较
* MyISAM的data域存放的是数据记录的地址。
* InnoDB的数据文件本身就是索引文件。
* InnoDB的辅助索引data域存储相应记录主键的值而不是地址。

### Mysql事务的基本要素（ACID）
* 原子性（Atomicity）
* 一致性（Consistency）
* 隔离性（Isolation）
* 持久性（Durability）

### Mysql事务的并发
* 脏读：一个事务读取了另一个事务未提交的数据。
* 不可重复读：一个事务读取同一行数据，多次读取结果不同。
* 幻读：一个事务读取到了别的事务插入的数据。

### Mysql分布式事务
CAP定理又称CAP原则，指的是在一个分布式系统中，Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），最多只能同时三个特性中的两个，三者不可兼得。

### Mysql Having和Where的区别
* having是将所有数据读入内存，在分组统计前，根据having的条件再将不符合条件的数据删除；where是数据从磁盘读入内存时候一条一条判断的。
* having可以使用字段别名；where不可用。
* having可以使用统计函数，where不可用。
* having筛选必须是根据前面select字段的值进行筛选。

### Mysql浮点数
* float单精度，4字节，32位；
* double双精度，8字节，64位；
* decimal小数值，官方唯一指定能精确存储的类型；

> 数据转成二进制后，只能截取前32位或64位进行存储，然后读取后将会出现误差。

### Mysql日志系统
* redo log: 重做日志，用于事务持久性，重启mysql服务时，根据redo log进行重做，从而达到事务的持久性这一特性。
* undo log: 回滚日志，用于事务回滚，保存事务发生之前的版本。
* bin  log: 二进制日志，用于数据库的基于时间点的还原。

## redis
### Redis缓存问题
* 缓存穿透：查询一个一定不存在的数据。
* 缓存击穿：大量的请求同时查询一个key时，此时这个key正好失效了。
* 缓存雪崩：当某一时刻发生大规模的缓存失效的情况。

### Redis哨兵模式
* 集群监控：负责监控Redis master和slave进程是否正常工作；
* 消息通知：如果某个Redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员；
* 故障转移：如果master node挂掉了，会自动转移到slave node上；
* 配置中心：如果故障转移发生了，通知client客户端新的master地址。
> 哨兵的作用就是监控redis主、从数据库是否正常运行，主出现故障自动将从数据库转换为主数据库。

### Redis集群模式
cluster可以说是sentinel和主从模式的结合体，通过cluster可以实现主从和master重选功能。

### 单线程的Redis为什么这么快
* 纯内存操作
* 单线程操作，避免了频繁的上下文切换
* 采用了非阻塞I/O多路复用机制

### Redis的过期策略
redis采用的是定期删除+惰性删除策略。

> 产生问题：如果定期删除没删除key。然后你也没及时去请求key，也就是说惰性删除也没生效。这样，redis的内存会越来越高。那么就应该采用内存淘汰机制。

## elastic search
### ik分词模式
ik分词器有两种分词模式：ik_max_word和ik_smart模式。

* 会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、华人、人民共和国、人民、共和国、大会堂、大会、会堂等词语。
* 会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。

### 倒排索引
倒排索引也叫反向索引，通俗来讲正向索引是通过key找value，反向索引则是通过value找key。

#### term index
就像字典里的索引页一样，先快速定位offset，然后通过顺序查找，结合FST(Finite State Transducers)的压缩技术，term index以树的形式缓存在内存中。

#### term dictionary
就像字典里的字典一样，将所有的term排个序，二分法查找term，logN的查找效率。
