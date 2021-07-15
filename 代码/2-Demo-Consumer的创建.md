# Consumer的创建

## 异步producer

1）导入依赖

```xml
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>0.11.0.0</version>
</dependency>
```

2）消费者创建

### 自动提交（同步提交）

```java
package com.atguigu.kafka;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Arrays;
import java.util.Properties;

public class CustomConsumer {
public static void main(String[] args) {
     Properties props = new Properties();
     props.put("bootstrap.servers", "hadoop102:9092");
     props.put("group.id", "test");
    // 是否自动提交
     props.put("enable.auto.commit", "true");
     props.put("auto.commit.interval.ms", "1000");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     // 订阅first topic的消息
     consumer.subscribe(Arrays.asList("first"));
     while (true) {
         // poll后面是timeout，如果轮询为空就等待100ms
         ConsumerRecords<String, String> records = consumer.poll(100);
         // 一次拉取会获得多个值 所以需要for遍历
         for (ConsumerRecord<String, String> record : records)
     		System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
     }
 }
}

```

当消费者没有offset或者offset所标记的文件已经经过7天被删除，就能通过以下方法获得offset

```java
AUTO_OFFSET_RESET_DOC = "earliest"; //获取最早的offset，即文件中最早保存的offset 即from beginning
AUTO_OFFSET_RESET_DOC = "latest"; // 获取最新的offset，不再管之前的消息，从开启消费者开始获取数据
```

### 手动提交

虽然自动提交 offset 十分简介便利，但由于其是基于时间提交的，开发人员难以把握 offset 提交的时机。因此 Kafka 还提供了手动提交 offset 的 API。 手动提交 offset 的方法有两种：**分别是 commitSync（同步提交）和 commitAsync（异步 提交）**。

两者的相同点是，都会将本次 poll 的一批数据最高的偏移量提交；

不同点是， commitSync 阻塞当前线程，一直到提交成功，**并且会自动失败重试（由不可控因素导致， 也会出现提交失败）**；而 commitAsync 则**没有失败重试机制，故有可能提交失败**。

#### 同步提交

```java
package com.atguigu.kafka.consumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import java.util.Arrays;
import java.util.Properties;
public class CustomComsumer {
 public static void main(String[] args) {
     Properties props = new Properties();
    //Kafka 集群
     props.put("bootstrap.servers", "hadoop102:9092");
    //消费者组，只要 group.id 相同，就属于同一个消费者组
     props.put("group.id", "test");
     props.put("enable.auto.commit", "false");//关闭自动提交 offset
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     consumer.subscribe(Arrays.asList("first"));//消费者订阅主题
     while (true) {
        //消费者拉取数据
        ConsumerRecords<String, String> records =
        consumer.poll(100);
         for (ConsumerRecord<String, String> record : records) {
             System.out.printf("offset = %d, key = %s, value
            = %s%n", record.offset(), record.key(), record.value());
     	}
        //同步提交，当前线程会阻塞直到 offset 提交成功 
         consumer.commitSync();
     }
 }
}
```

#### 异步提交

```java
package com.atguigu.kafka.consumer;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;
import java.util.Arrays;
import java.util.Map;
import java.util.Properties;
public class CustomConsumer {
 public static void main(String[] args) {
     Properties props = new Properties();
     //Kafka 集群
     props.put("bootstrap.servers", "hadoop102:9092");
     //消费者组，只要 group.id 相同，就属于同一个消费者组
     props.put("group.id", "test");

     //关闭自动提交 offset
     props.put("enable.auto.commit", "false");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     consumer.subscribe(Arrays.asList("first"));//消费者订阅主题
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);//消费者拉取数据
         for (ConsumerRecord<String, String> record : records) {
             System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
         }
        //异步提交
         consumer.commitAsync(new OffsetCommitCallback() {
         @Override
         public void onComplete(Map<TopicPartition,
            OffsetAndMetadata> offsets, Exception exception) {
            if (exception != null) {
                System.err.println("Commit failed for" + offsets);
            }
         }
     	});
     }
	}
}
```

3） 数据漏消费和重复消费分析 无论是同步提交还是异步提交 offset，都有可能会造成数据的漏消费或者重复消费。先 提交 offset 后消费，有可能造成数据的漏消费；而先消费后提交 offset，有可能会造成数据 的重复消费。

#### 自定义offset

当消费者组里面有新的人订阅主题，或者有的消费者推出消费者组，那么就得对分区进行重新分配。所以我们需要创建一个例如mysql的数据库，记录每个分区的offset，然后在重新分配分区的时候（rebalance）的时候。实现ConsumerRebalanceListener接口，重写 onPartitionsRevoked和onPartitionsAssigned来进行offset的获取和上传。

```java
package com.atguigu.kafka.consumer;
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;
import java.util.Arrays;
import java.util.Map;
import java.util.Properties;
public class CustomConsumer {
 public static void main(String[] args) {
     Properties props = new Properties();
     //Kafka 集群
     props.put("bootstrap.servers", "hadoop102:9092");
     
     //消费者组，只要 group.id 相同，就属于同一个消费者组
     props.put("group.id", "test");

     //关闭自动提交 offset
     props.put("enable.auto.commit", "false");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     //消费者订阅主题
     consumer.subscribe(Arrays.asList("first"), new ConsumerRebalanceListener() {

         //该方法会在 Rebalance 之前调用
         @Override
         public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
         	commitOffset(currentOffset);
         }
         //该方法会在 Rebalance 之后调用
         @Override
         public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
             currentOffset.clear();
             for (TopicPartition partition : partitions) {
                consumer.seek(partition, getOffset(partition));// 定位到最近提交的 offset 位置继续消费
             }
     	}
     });
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);//消费者拉取数据
         for (ConsumerRecord<String, String> record : records) {
            System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
            currentOffset.put(new TopicPartition(record.topic(), record.partition()), record.offset());
         }
         commitOffset(currentOffset);//异步提交
         }
     }
     //获取某分区的最新 offset
     private static long getOffset(TopicPartition partition) {
     return 0;
     }
     //提交该消费者所有分区的 offset
     private static void commitOffset(Map<TopicPartition, Long> currentOffset) {
     }
}

```

