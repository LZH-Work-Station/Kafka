#  数据可靠性保证

## ISR

Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集 合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower 长时间 未 向 leader 同 步 数 据 ， 则 该 follower 将 被 踢 出 ISR ， 该 时 间 阈 值 由 尚硅谷大数据技术之 Kafka Leader 发生故障之后，就会从 ISR 中选举新的 leader。

## ACKS的参数配置

- acks = 0

producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还 没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据



- acks = 1

producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据

<img src="E:\Javadream\Kafka\6-Kafka 数据可靠性保证.assets\image-20210712151033276.png" alt="image-20210712151033276" style="zoom:200%;" />

leader返回ack，isr里面的follower还没来得及落盘leader就down掉了

- acks = -1

必须等待所有isr里面的follower更新完毕才能返回ack，如果leader半截down掉，可能会造成 数据的重复，比如leader写完12345，follower写完123，leader down掉，follower成为新leader，由于长时间没有ack，producer重发刚才的12345，这样123就重复了

<img src="E:\Javadream\Kafka\6-Kafka 数据可靠性保证.assets\image-20210712151136655.png" alt="image-20210712151136655" style="zoom:200%;" />

## 故障处理细节

![image-20210712151345171](E:\Javadream\Kafka\6-Kafka 数据可靠性保证.assets\image-20210712151345171.png)

**LEO：指的是每个副本最大的 offset； **

**HW：指的是消费者能见到的最大的 offset，ISR 队列中最小的 LEO。** 

（1）follower 故障 follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower 会读取本地磁盘 记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。 等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重 新加入 ISR 了。 

（2）leader 故障 leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性，其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader 同步数据。 注意：这只能保证副本之间的数据一致性，**并不能保证数据不丢失或者不重复。**

**步骤：**

ISR中选取新的leader，然后其他所有follower都把数据删除到HW位置，然后向着新的follower同步。例如Leader：12345

follower1：12

follower2：1234

follower3：12345

这时HW=2，当leader down了，follower2加入当选leader，然后follower3先将345删掉为了同步到HW，再根据follower2的数据同步34，follower1由于已经是HW，所以不存在删除，直接同步34.

**所以不能保证数据不丢失，因为5就丢失了，而且由于没有ACK，当producer重传的时候会出现1234数据的重复**

## Exactly Once

Producer在初始化的时候会有一个PID，然后PID，Partition和SeqNumber会组成消息的key，每个broker只会缓存key不同的消息，这样就保证了消息的不重复性



但是，如果producer重启，PID被重新分配，这样就会造成，key不同了，也就无法保证exactly Once。
