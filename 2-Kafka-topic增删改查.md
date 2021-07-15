# Kafka-topic增删改查

## 进入kafka的bin目录，使用kafka-topics.sh来增删改查

kafka依赖于zookeeper来帮助我们进行存储

#### 如果出现问题看日志，日志在logs的server.log里面

### 创建

partition是把first topic的分区，replication是有多少个数据副本（replication-factor 2代表备份一份）**副本数不能超过集群的机器数**

![image-20210711202641414](E:\Javadream\Kafka\Kafka-topic增删改查.assets\image-20210711202641414.png)

### 查看

![image-20210711202703590](E:\Javadream\Kafka\Kafka-topic增删改查.assets\image-20210711202703590.png)

### 删除

![image-20210711203511810](E:\Javadream\Kafka\Kafka-topic增删改查.assets\image-20210711203511810.png)

需要设置delete.topic.enable为true才能删除

### Describe

![image-20210711213512723](E:\Javadream\Kafka\Kafka-topic增删改查.assets\image-20210711213512723.png)

