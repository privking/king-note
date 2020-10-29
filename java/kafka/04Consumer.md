# Consumer

## SimpleConsumer

```java
public class SimpleConsumer {
    public static void main(String[] args) {
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //自动提交 offset延迟
        properties.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG,"1000");
        //开启自动提交
        properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        //反序列化key
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        //反序列化 value
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringDeserializer");
        //消费者组
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "bigdataGroup");
        //启动消费者后 如果offset不存在从哪里开始消费
        //earliest 最开始
        //latest 最后（默认）
        //none 没有找到offset时 抛异常
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        //订阅主题 可以订阅多个主题
        consumer.subscribe(Arrays.asList("hello"));
        //拉取

        while(true){
            ConsumerRecords<String, String> poll = consumer.poll(Duration.ofSeconds(1));

            poll.forEach(obj->{
                System.out.println("obj.headers() = " + obj.headers());
                System.out.println("obj.key() = " + obj.key());
                System.out.println("obj.offset() = " + obj.offset());
                System.out.println("obj.partition() = " + obj.partition());
                System.out.println("obj.value() = " + obj.value());

            });
        }

    }
}
```

## NoAutoCommitConsumer

```java
/**
* 不使用自动提交offset
* properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
* 
*/
public class NoAutoCommitConsumer {
    public static void main(String[] args) {
        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //自动提交 offset延迟
        //properties.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG,"1000");
        //开启自动提交
        properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        //反序列化key
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        //反序列化 value
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringDeserializer");
        //消费者组
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "bigdataGroup");
        //启动消费者后 如果offset不存在从哪里开始消费
        //earliest 最开始
        //latest 最后（默认）
        //none 没有找到offset时 抛异常
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        //订阅主题 可以订阅多个主题
        consumer.subscribe(Arrays.asList("hello"));
        while(true){
            ConsumerRecords<String, String> poll = consumer.poll(Duration.ofSeconds(1));

            poll.forEach(obj->{
                System.out.println("obj.headers() = " + obj.headers());
                System.out.println("obj.key() = " + obj.key());
                System.out.println("obj.offset() = " + obj.offset());
                System.out.println("obj.partition() = " + obj.partition());
                System.out.println("obj.value() = " + obj.value());

            });
            //在提交前挂掉了  没有提交 就会出现重复

            //异步提交
            //consumer.commitAsync();
            consumer.commitAsync((offsets,e)->{
                if(e==null){
                    System.out.println("offsets.keySet() = " + offsets.keySet());

                }else{
                    System.out.println(e.getMessage());
                }

            });
            //同步提交
            //consumer.commitSync();

        }
    }
}
```

## CustomOffsetConsumer

```java
/*
* 自己维护offset
* 因为是一批一批拉取数据 异步提交和同步提交都可能出现问题
* 通常可以维护在mysql 用事务绑定
*/
public class CustomConsumer {
    //解决自动提交的问题
    //自定义将offset维护在mysql
    public static void main(String[] args) {
        Map<TopicPartition, Long> currentOffset = new HashMap<>();

        Properties properties = new Properties();
        //指定连接kafka集群
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"node1:9092,node2:9092,node3:9092");
        //自动提交 offset延迟
        //properties.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG,"1000");
        //开启自动提交
        properties.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        //反序列化key
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        //反序列化 value
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringDeserializer");
        //消费者组
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "bigdataGroup");
        //启动消费者后 如果offset不存在从哪里开始消费
        //earliest 最开始
        //latest 最后（默认）
        //none 没有找到offset时 抛异常
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);

        //订阅主题 可以订阅多个主题
        consumer.subscribe(Arrays.asList("hello"), new ConsumerRebalanceListener() {
            //在重新划分partitions（rebalance）前， KafkaConsumer.close(Duration) KafkaConsumer.unsubscribe()时会调用
            //新的consumer一定会在旧的consumer调用后调用
            //主要用来提交offset 或者是提交刷新自己的自定义缓存
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                commitOffset(currentOffset);
            }

            //分区发生变化时调用
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                currentOffset.clear();
                for (TopicPartition partition : partitions) {
                    consumer.seek(partition, getOffset(partition));//定位到最近提交的offset位置继续消费
                }
            }
        });

        //拉取

        while(true){
            ConsumerRecords<String, String> poll = consumer.poll(Duration.ofSeconds(1));

               try{
                   for (ConsumerRecord<String, String> record : poll) {
                        //更新currentOffset
                   }
               }finally {
                   commitOffset(currentOffset);//异步提交
               }
        }
    }

    //获取某分区的最新offset
    private static long getOffset(TopicPartition partition) {
        //select from mysql
        return 0;
    }

    //提交该消费者所有分区的offset
    private static void commitOffset(Map<TopicPartition, Long> currentOffset) {
        //update mysql
    }

}
```

## ConsumerInterceptor

```java
public class MyConsumerInterceptor implements ConsumerInterceptor {
    //可以修改records
    //一个主要用例是让第三方组件连接到消费者应用程序中，以进行自定义监控、日志记录等。
    @Override
    public ConsumerRecords onConsume(ConsumerRecords records) {
        return null;
    }

    @Override
    public void close() {

    }
    //提交offset后调用
    @Override
    public void onCommit(Map offsets) {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```