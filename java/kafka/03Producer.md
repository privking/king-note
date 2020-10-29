# Producer

Kafka的Producer发送消息采用的是**异步发送**的方式。在消息发送的过程中，涉及到了两个线程——main线程和Sender线程，以及一个线程共享变量——RecordAccumulator。main线程将消息发送给RecordAccumulator，Sender线程不断从RecordAccumulator中拉取消息发送到Kafka broker。

![image-20201027234214723](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201027234214723-1603813341-d2d0d1.png)

## 依赖

```xml
<dependency>
<groupId>org.apache.kafka</groupId>
<artifactId>kafka-clients</artifactId>
<version>same with kafka</version>
</dependency>
```

## SimpleProducer

```java
public class SimpleProducer {
    public static void main(String[] args) {
        //创建kafka配置信息
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //ack应答级别 -1/all 0 1
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        //重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 3);
        //批次大小 16k   字符串key都可以通过ProducerConfig获取到 也可以自定义，重写过滤器分区器时可获得
        properties.put("batch.size", 16384);
        //等待时间 1ms
        properties.put("linger.ms", 1);
        //16k或1ms就会提交到RecordAccumulator

        //RecordAccumulator缓冲区大小 32M
        properties.put("buffer.memory", 33554432);
        //key序列化
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        //value序列化
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        //创建producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        for (int i = 0; i < 10; i++) {
            //该发送为异步发送 
            //send返回为future  
            //.get()即可变成同步
            producer.send(new ProducerRecord<String, String>("hello","producer1"+i));
        }
       producer.close();
    }
}
```

## CallBackProducer

```java
/**
有回调的send
*/
public class CallBackProducer {
    public static void main(String[] args) {

        //创建kafka配置信息
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //ack应答级别 -1/all 0 1
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        //重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 3);
        //批次大小 16k
        properties.put("batch.size", 16384);
        //等待时间 1ms
        properties.put("linger.ms", 1);
        //16k或1ms就会提交到RecordAccumulator

        //RecordAccumulator缓冲区大小 32M
        properties.put("buffer.memory", 33554432);
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        //创建producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        for (int i = 0; i < 10; i++) {
            final int finalI = i;
            //send 返回值为future  .get()即同步发送
            producer.send(new ProducerRecord<String, String>("hello","key", "producer2" + i), (recordMetadata, e) -> {
                if(e==null){
                    System.out.println(finalI);
                    System.out.println(recordMetadata.partition());
                    System.out.println(recordMetadata.topic());
                    System.out.println(recordMetadata);
                    System.out.println("-----------");
                }else{
                    e.printStackTrace();
                }
            });
        }
        producer.close();

    }
}
```

## PartitionProducer

```java
/**
 * 自定义发送到哪个分区 
 * 通过 properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"priv.king.partitioner.MyPartitioner");
 * 默认是key Hash(不完全)
 **/
public class PartitionProducer {
    public static void main(String[] args) {
        //创建kafka配置信息
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //ack应答级别 -1/all 0 1
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        //重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 3);
        //批次大小 16k
        properties.put("batch.size", 16384);
        //等待时间 1ms
        properties.put("linger.ms", 1);
        //16k或1ms就会提交到RecordAccumulator

        //RecordAccumulator缓冲区大小 32M
        properties.put("buffer.memory", 33554432);
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"priv.king.partitioner.MyPartitioner");

        //创建producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        for (int i = 0; i < 10; i++) {
            final int finalI = i;
            producer.send(new ProducerRecord<String, String>("hello", "producer2" + i), new Callback() {
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    if(e==null){
                        System.out.println(finalI);
                        System.out.println(recordMetadata.partition());
                        System.out.println(recordMetadata.topic());
                        System.out.println(recordMetadata.partition());
                        System.out.println("-----------");
                    }else{
                        e.printStackTrace();
                    }


                }
            });
        }
        producer.close();
    }
}



public class MyPartitioner implements Partitioner {
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        //cluster.partitionCountForTopic(topic);
        //cluster.partitionsForTopic(topic);
        return 0;
    }

    public void close() {

    }

    public void configure(Map<String, ?> configs) {
        System.out.println(configs);

        //configs   在producer中传入的Properties
    }
}
```

## InterceptorProducer

```java
/**
 * 自定义过滤器
 * 通过 properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, Arrays.asList("priv.king.interceptor.TimeInterceptor"));
 * 
 **/
public class InterceptorProducer {
    public static void main(String[] args) {

        //创建kafka配置信息
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        //ack应答级别 -1/all 0 1
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        //重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 3);
        //批次大小 16k
        properties.put("batch.size", 16384);
        //等待时间 1ms
        properties.put("linger.ms", 1);
        //16k或1ms就会提交到RecordAccumulator

        //RecordAccumulator缓冲区大小 32M
        properties.put("buffer.memory", 33554432);
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        
        properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, Arrays.asList("priv.king.interceptor.TimeInterceptor"));
        //创建producer
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        for (int i = 0; i < 10; i++) {
            producer.send(new ProducerRecord<String, String>("hello", "producer1" + i));
        }
        producer.close();

    }
}


public class TimeInterceptor implements ProducerInterceptor<String,String> {

    Long countSuccess=0L;
    Long countError=0L;
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        String value = record.value();
        value+= ","+System.currentTimeMillis();

        ProducerRecord<String, String> record1 = new ProducerRecord<>(record.topic(), record.key(), value);
        return record1;
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        //响应ack时调用
        if(exception==null){
            countSuccess++;
        }else{
            countError++;
        }
    }

    @Override
    public void close() {
        //This is called when interceptor is closed
        //在producer.close()是调用
        System.out.printf("success:%d,error:%d\n",countSuccess,countError);
    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```