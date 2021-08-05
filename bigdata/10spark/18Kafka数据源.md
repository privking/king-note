# Kafka数据源

## 概念

### 基于接收器方式

这种方法使用接收器来接收数据。Receiver 是使用 Kafka 高级消费者 API 实现的。与所有接收器一样，通过接收器从 Kafka 接收到的数据存储在 Spark 执行器中，然后由 Spark Streaming 启动的作业处理数据。

在默认配置下，这种方法可能会在失败时丢失数据，为了确保零数据丢失，您必须在 Spark Streaming 中额外启用 Write Ahead Logs，这会同步保存所有收到的 Kafka将数据写入分布式文件系统（例如 HDFS）上的预写日志，以便所有数据可以在失败时恢复

**Kafka 中的主题分区与 Spark Streaming 中生成的 RDD 的分区无关**。因此，增加主题特定分区`KafkaUtils.createStream()`的数量**只会增加使用单个接收器内使用的主题的线程数量**。**它不会增加 Spark 在处理数据时的并行度**。向吧数据从kafka各个分区中union到接收器上，再进行处理

### 直接流方式

这种方法不是使用接收器来接收数据，而是定期向 Kafka 查询每个主题+分区中的最新偏移量，并相应地定义每个批次中要处理的偏移量范围。

*简化的并行性：*无需创建多个输入 Kafka 流并将它们联合起来。使用`directStream`，**Spark Streaming 将创建与要使用的 Kafka 分区一样多的 RDD 分区**，所有这些分区都将从 Kafka 并行读取数据。所以Kafka和RDD分区之间存在一对一的映射关系，更容易理解和调优。

*效率：*在第一种方法中实现零数据丢失需要将数据存储在预写日志中，从而进一步复制数据。这实际上是低效的，因为数据被有效地复制了两次——一次由 Kafka 复制，第二次由预写日志复制。第二种方法消除了这个问题，因为没有接收器，因此不需要预写日志。**只要你有足够的 Kafka 保留，就可以从 Kafka 恢复消息。**

*恰好一次语义：*第一种方法使用 Kafka 的高级 API 在 Zookeeper 中存储消耗的偏移量。这是传统上从 Kafka 消费数据的方式。虽然这种方法（结合预写日志）可以确保零数据丢失（即至少一次语义），但在某些故障下，某些记录可能会被消耗两次。这是因为 Spark Streaming 可靠接收的数据与 Zookeeper 跟踪的偏移量之间存在不一致。因此，在第二种方法中，我们使用不使用 Zookeeper 的简单 Kafka API。偏移量由 Spark Streaming 在其检查点内跟踪。这消除了 Spark Streaming 和 Zookeeper/Kafka 之间的不一致，因此即使出现故障，Spark Streaming 也能有效地接收每条记录一次。

请注意，这种方法的一个缺点是它不会更新 Zookeeper 中的偏移量，因此基于 Zookeeper 的 Kafka 监控工具不会显示进度。

## 实现方式（直接流）

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
    <version>2.1.1</version>
</dependency>

```

### 每次从最新开始消费

```scala
object HighKafka {
    def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("HighKafka")
        val ssc = new StreamingContext(conf, Seconds(3))
        
        // kafka 参数
        //kafka参数声明
        val brokers = "hadoop201:9092,hadoop202:9092,hadoop203:9092"
        val topic = "first"
        val group = "bigdata"
        val kafkaParams = Map(
            ConsumerConfig.GROUP_ID_CONFIG -> group,
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokers,
         )
        val dStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](
            ssc, kafkaParams, Set(topic))
        
        dStream.print()
        
        ssc.start()
        ssc.awaitTermination()
    }
}

```

### checkpoint

能够接着上一次消费，但是会产生很多小文件

```scala
object HighKafka2 {
    
    def createSSC(): StreamingContext = {
        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("HighKafka")
        val ssc = new StreamingContext(conf, Seconds(3))
        // 偏移量保存在 checkpoint 中, 可以从上次的位置接着消费
        ssc.checkpoint("./ck1")
        // kafka 参数
        //kafka参数声明
        val brokers = "hadoop201:9092,hadoop202:9092,hadoop203:9092"
        val topic = "first"
        val group = "bigdata"
        val kafkaParams = Map(
            ConsumerConfig.GROUP_ID_CONFIG -> group,
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokers
        )
        val dStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](
            ssc, kafkaParams, Set(topic))
        
        dStream.print()
        ssc
    }
    
    def main(args: Array[String]): Unit = {
        
        val ssc: StreamingContext = StreamingContext.getActiveOrCreate("./ck1", () => createSSC())
        ssc.start()
        ssc.awaitTermination()
    }
}

```

### 自己维护offset

```scala
object LowKafka {
    // 获取 offset
    def getOffset(kafkaCluster: KafkaCluster, group: String, topic: String): Map[TopicAndPartition, Long] = {
        // 最终要返回的 Map
        var topicAndPartition2Long: Map[TopicAndPartition, Long] = Map[TopicAndPartition, Long]()
        
        // 根据指定的主体获取分区信息
        val topicMetadataEither: Either[Err, Set[TopicAndPartition]] = kafkaCluster.getPartitions(Set(topic))
        // 判断分区是否存在
        if (topicMetadataEither.isRight) {
            // 不为空, 则取出分区信息
            val topicAndPartitions: Set[TopicAndPartition] = topicMetadataEither.right.get
            // 获取消费消费数据的进度
            val topicAndPartition2LongEither: Either[Err, Map[TopicAndPartition, Long]] =
                kafkaCluster.getConsumerOffsets(group, topicAndPartitions)
            // 如果没有消费进度, 表示第一次消费
            if (topicAndPartition2LongEither.isLeft) {
                // 遍历每个分区, 都从 0 开始消费
                topicAndPartitions.foreach {
                    topicAndPartition => topicAndPartition2Long = topicAndPartition2Long + (topicAndPartition -> 0)
                }
            } else { // 如果分区有消费进度
                // 取出消费进度
                val current: Map[TopicAndPartition, Long] = topicAndPartition2LongEither.right.get
                topicAndPartition2Long ++= current
            }
        }
        // 返回分区的消费进度
        topicAndPartition2Long
    }
    
    // 保存消费信息
    def saveOffset(kafkaCluster: KafkaCluster, group: String, dStream: InputDStream[String]) = {
        
        dStream.foreachRDD(rdd => {
            var map: Map[TopicAndPartition, Long] = Map[TopicAndPartition, Long]()
            // 把 RDD 转换成HasOffsetRanges对
            val hasOffsetRangs: HasOffsetRanges = rdd.asInstanceOf[HasOffsetRanges]
            // 得到 offsetRangs
            val ranges: Array[OffsetRange] = hasOffsetRangs.offsetRanges
            ranges.foreach(range => {
                // 每个分区的最新的 offset
                map += range.topicAndPartition() -> range.untilOffset
            })
            kafkaCluster.setConsumerOffsets(group,map)
        })
    }
    
    def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("HighKafka")
        val ssc = new StreamingContext(conf, Seconds(3))
        // kafka 参数
        //kafka参数声明
        val brokers = "hadoop201:9092,hadoop202:9092,hadoop203:9092"
        val topic = "first"
        val group = "bigdata"
        val kafkaParams = Map(
            ConsumerConfig.GROUP_ID_CONFIG -> group,
            ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokers
        )
        // 读取 offset
        val kafkaCluster = new KafkaCluster(kafkaParams)
        val fromOffset: Map[TopicAndPartition, Long] = getOffset(kafkaCluster, group, topic)
        val dStream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder, String](
            ssc,
            kafkaParams,
            fromOffset,
            (message: MessageAndMetadata[String, String]) => message.message()
        )
        dStream.print()
        // 保存 offset
        saveOffset(kafkaCluster, group, dStream)
        ssc.start()
        ssc.awaitTermination()
    }
}

```

