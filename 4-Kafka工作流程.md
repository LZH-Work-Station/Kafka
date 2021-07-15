# Kafka工作流程

## 1. Kafka工作流程及文件存储机制

![image-20210712144714142](E:\Javadream\Kafka\4-Kafka工作流程.assets\image-20210712144714142.png)

Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic 的。 topic 是逻辑上的概念，而 partition 是物理上的概念，每个 partition 对应于一个 log 文 件，该 log 文件中存储的就是 producer 生产的数据。Producer 生产的数据会被不断追加到该 log 文件末端，且每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己 消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费。

## 2.Kafka文件存储机制

每个topic在每个分区都有一个log文件记录该分区Topic数据。每个log文件有很多个segment，被分成了很多片。每一条消息就对应一片，然后这条消息的起始位置会被记录在.index文件中。当消费者通过之前的offset，比如想要找到消息3，然后就会 先查找消息3 在哪个segment，然后找到对应的 在这个segment中的哪个对应的起始位置，最后在log位置定位到起始位置然后进行新的消费

![image-20210712144841325](E:\Javadream\Kafka\4-Kafka工作流程.assets\image-20210712144841325.png)

![image-20210712144920668](E:\Javadream\Kafka\4-Kafka工作流程.assets\image-20210712144920668.png)

