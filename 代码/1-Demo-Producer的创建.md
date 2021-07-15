# producer的创建

## 异步producer

1）导入依赖

```xml
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>0.11.0.0</version>
</dependency>
```

2) 有回调的话需要实现callback()接口放进producer.send里面，没有回调直接 send 就完事了

```java
producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), Integer.toString(i))
```



```java
package com.atguigu.kafka;
import org.apache.kafka.clients.producer.*;
import java.util.Properties;
import java.util.concurrent.ExecutionException;
public class CustomProducer {
 public static void main(String[] args) throws ExecutionException, InterruptedException {
     Properties props = new Properties();
     //kafka 集群，broker-list
     props.put("bootstrap.servers", "hadoop102:9092");
     props.put("acks", "all");
     //重试次数
     props.put("retries", 1);
     //批次大小
     props.put("batch.size", 16384);
     //等待时间
     props.put("linger.ms", 1);
     //RecordAccumulator 缓冲区大小
     props.put("buffer.memory", 33554432);
     props.put("key.serializer",
    		"org.apache.kafka.common.serialization.StringSerializer");
     props.put("value.serializer",
    		"org.apache.kafka.common.serialization.StringSerializer");
     Producer<String, String> producer = new KafkaProducer<>(props);

     for (int i = 0; i < 100; i++) {
        // producer有多种发送方式，就向我们之前介绍的，可以指定分区，可以K,V形式然后根据K的hash值分区，或者直接传入Value，这里面就用的 K,V形式，可以通过回调函数metadata.partitions()来查看分区
     	producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), 	Integer.toString(i)), 
             (metadata, exception) -> {
                 if (exception == null) {
                     System.out.println("success->" + metadata.offset());
                 } else {
                    exception.printStackTrace();
                 }
         	}
     	});
     }
     producer.close();
 	 }
}

```

## 同步发送

即阻塞形式发送，没有acks就不会发下一段消息

producer.send()会返回一个future对象，就是多线程里面的哪个future，然后.get（）可以获得future的异常和返回值，同时阻塞程序

```java
package com.atguigu.kafka;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;
import java.util.concurrent.ExecutionException;
public class CustomProducer {
public static void main(String[] args) throws ExecutionException,
InterruptedException {
     Properties props = new Properties();
     props.put("bootstrap.servers", "hadoop102:9092");//kafka 集
    群，broker-list
     props.put("acks", "all");
     props.put("retries", 1);//重试次数
     props.put("batch.size", 16384);//批次大小
     props.put("linger.ms", 1);//等待时间
     props.put("buffer.memory", 33554432);//RecordAccumulator 缓
    冲区大小
     props.put("key.serializer",
    "org.apache.kafka.common.serialization.StringSerializer");
     props.put("value.serializer",
    "org.apache.kafka.common.serialization.StringSerializer");
     Producer<String, String> producer = new KafkaProducer<>(props);
     for (int i = 0; i < 100; i++) {
     	producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), Integer.toString(i))).get();
     }
     producer.close();
     }
}
```

