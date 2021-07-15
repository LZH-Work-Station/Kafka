# Zookeeper 在 Kafka 中的作用

Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所 有 topic 的分区副本分配和 leader 选举等工作。 Controller 的管理工作都是依赖于 Zookeeper 的。 以下为 partition 的 leader 选举过程：



![image-20210712154004159](E:\Javadream\Kafka\8-Zookeeper在Kafka中的作用.assets\image-20210712154004159.png)

## producer事务

为了实现跨分区跨会话的事务，需要引入一个全局唯一的 Transaction ID，并将 Producer 获得的PID 和Transaction ID 绑定。这样当Producer 重启后就可以通过正在进行的 Transaction ID 获得原来的 PID。 

为了管理 Transaction，Kafka 引入了一个新的组件 Transaction Coordinator。Producer 就 是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态。

Transaction Coordinator 还负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于 事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。

这样的话，即使producer 重启了，也能获取到上一个 PID，继而不会造成数据重复。

