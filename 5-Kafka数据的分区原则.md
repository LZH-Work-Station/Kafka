# 分区策略

![image-20210712150746252](E:\Javadream\Kafka\5-Kafka数据的分区原则.assets\image-20210712150746252.png)

（1）指明 partition 的情况下，直接将指明的值直接作为 partiton 值； 

（2）没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 			值； 

（3）既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后 面每次调用在这个整数上		自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。

