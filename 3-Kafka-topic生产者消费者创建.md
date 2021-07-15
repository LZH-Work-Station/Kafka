# Kafka-topic生产者消费者创建

## 1. 创建生产者

在小箭头后面输入东西 在消费者那管理就可以看到

![image-20210711214454127](E:\Javadream\Kafka\Kafka-topic生产者消费者创建.assets\image-20210711214454127.png)

## 2. 创建消费者

![image-20210711214700674](E:\Javadream\Kafka\Kafka-topic生产者消费者创建.assets\image-20210711214700674.png)

可以接收到消费者的消息



![image-20210711214751133](E:\Javadream\Kafka\Kafka-topic生产者消费者创建.assets\image-20210711214751133.png)

- 使用 --from-beginning能拿到 在这个consumer没有up之前的product的数据

## 3. 默认偏移文件

老版本 Consumer 的位移管理是依托于 Apache ZooKeeper 的，它会自动或手动地将位移数据提交到 ZooKeeper 中保存。当 Consumer 重启后，它能自动从 ZooKeeper 中读取位移数据，从而在上次消费截止的地方继续消费。这种设计使得 Kafka Broker 不需要保存位移数据，减少了 Broker 端需要持有的状态空间，因而有利于实现高伸缩性。
新版本 Consumer 的位移管理机制其实也很简单，就是**将 Consumer 的位移数据作为一条条普通的 Kafka 消息，提交到 __consumer_offsets 中。可以这么说，__consumer_offsets 的主要作用是保存 Kafka 消费者的位移信息。**它要求这个提交过程不仅要实现高持久性，还要支持高频的写操作。显然，Kafka 的主题设计天然就满足这两个条件，因此，使用 Kafka 主题来保存位移这件事情，实际上就是一个水到渠成的想法了。

- 后面的数字是consumer_offsets的分区，每次消费了数据就会在（可能是hash运算）所对应的分区写上消费到了多少偏移量

![image-20210711220030995](E:\Javadream\Kafka\3-Kafka-topic生产者消费者创建.assets\image-20210711220030995.png)
