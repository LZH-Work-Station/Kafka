# 自定义 Interceptor

对于 producer 而言，interceptor 使得用户在消息发送前以及 producer 回调逻辑前有机会 对消息做一些**定制化需求，比如修改消息**等。同时，producer 允许用户指定多个 interceptor 按序作用于同一条消息从而形成一个拦截链(interceptor chain)。Intercetpor 的实现接口是 org.apache.kafka.clients.producer.ProducerInterceptor，其定义的方法包括： 

（1）configure(configs) 获取配置信息和初始化数据时调用

（2）onSend(ProducerRecord)： 该方法封装进 KafkaProducer.send 方法中，即它运行在用户主线程中。Producer 确保在 尚消息被序列化以及计算分区前调用该方法。用户可以在该方法中对消息做任何操作，但最好 保证不要修改消息所属的 topic 和分区，否则会影响目标分区的计算。 **更改数据的方法**

（3）onAcknowledgement(RecordMetadata, Exception)： 该方法会在消息从 RecordAccumulator 成功发送到 Kafka Broker 之后，或者在发送过程 中失败时调用。并且通常都是在 producer 回调逻辑触发之前。onAcknowledgement 运行在 producer 的 IO 线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢 producer 的消息 发送效率。 **来确定broker是否从recordAccmulator那里拿了数据**

（4）close： 关闭 interceptor，主要用于执行一些资源清理工作 如前所述，interceptor 可能被运行在多个线程中，因此在具体实现时用户需要自行确保 线程安全。另外倘若指定了多个 interceptor，则 producer 将按照指定顺序调用它们，并仅仅 是捕获每个 interceptor 可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中 要特别留意。

## 案例

```java
（1）增加时间戳拦截器
package com.atguigu.kafka.interceptor;
import java.util.Map;
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
public class TimeInterceptor implements ProducerInterceptor<String, String> {
    @Override
    public void configure(Map<String, ?> configs) {
    }
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
    // 创建一个新的 record，把时间戳写入消息体的最前部
        return new ProducerRecord(record.topic(), record.partition(), record.timestamp(), record.key(), System.currentTimeMillis() + "," + record.value().toString());
        }
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
    }
    @Override
    public void close() {
    }
}

```

```java
（2）统计发送消息成功和发送失败消息数，并在 producer 关闭时打印这两个计数器
package com.atguigu.kafka.interceptor;
import java.util.Map;
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
public class CounterInterceptor implements ProducerInterceptor<String, String>{
    private int errorCounter = 0;
    private int successCounter = 0;
	@Override
    public void configure(Map<String, ?> configs) {
    }

	@Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        return record;
    }
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        // 统计成功和失败的次数
         if (exception == null) {
         	successCounter++;
         } else {
         	errorCounter++;
         }
    }
    @Override
    public void close() {
     // 保存结果
     System.out.println("Successful sent: " + successCounter);
     System.out.println("Failed sent: " + errorCounter);
    }
}
```

```java
（3）producer 主程序
package com.atguigu.kafka.interceptor;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
public class InterceptorProducer {
    public static void main(String[] args) throws Exception {
        // 1 设置配置信息
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");
        props.put("acks", "all");
        props.put("retries", 3);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        // 2 构建拦截链
        List<String> interceptors = new ArrayList<>(); 
        interceptors.add("com.atguigu.kafka.interceptor.TimeInterceptor");
        interceptors.add("com.atguigu.kafka.interceptor.CounterInterceptor");

        props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,interceptors);

        String topic = "first";
        Producer<String, String> producer = new KafkaProducer<>(props);

        // 3 发送消息
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> record = new ProducerRecord<>(topic, "message" + i);
            producer.send(record);
        }
        // 4 一定要关闭 producer，这样才会调用 interceptor 的 close 方法
        producer.close();
	}
}
```

