# Source&Sink~V1.13

基于V1.13

## 容错保证

当程序出现错误的时候，Flink 的容错机制能恢复并继续运行程序。这种错误包括机器硬件故障、网络故障、瞬态程序故障等等。

只有当 source 参与了快照机制的时候，Flink 才能保证对自定义状态的精确一次更新。下表列举了 Flink 与其自带连接器的状态更新的保证。

|                       |                                      |                                   |
| :-------------------- | :----------------------------------- | :-------------------------------- |
| Source                | Guarantees                           | Notes                             |
| Apache Kafka          | 精确一次                             | 根据你的版本用恰当的 Kafka 连接器 |
| AWS Kinesis Streams   | 精确一次                             |                                   |
| RabbitMQ              | 至多一次 (v 0.10) / 精确一次 (v 1.0) |                                   |
| Twitter Streaming API | 至多一次                             |                                   |
| Google PubSub         | 至少一次                             |                                   |
| Collections           | 精确一次                             |                                   |
| Files                 | 精确一次                             |                                   |
| Sockets               | 至多一次                             |                                   |

为了保证端到端精确一次的数据交付（在精确一次的状态语义上更进一步），sink需要参与 checkpointing 机制。下表列举了 Flink 与其自带 sink 的交付保证（假设精确一次状态更新）

|                     |                     |                                            |
| :------------------ | :------------------ | :----------------------------------------- |
| Sink                | Guarantees          | Notes                                      |
| Elasticsearch       | 至少一次            |                                            |
| Kafka producer      | 至少一次 / 精确一次 | 当使用事务生产者时，保证精确一次 (v 0.11+) |
| Cassandra sink      | 至少一次 / 精确一次 | 只有当更新是幂等时，保证精确一次           |
| AWS Kinesis Streams | 至少一次            |                                            |
| File sinks          | 精确一次            |                                            |
| Socket sinks        | 至少一次            |                                            |
| Standard output     | 至少一次            |                                            |
| Redis sink          | 至少一次            |                                            |

## Kafka 

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.11</artifactId>
    <version>1.13.0</version>
</dependency>
```

### source

```scala
val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
properties.setProperty("group.id", "test")
val stream = env
    .addSource(new FlinkKafkaConsumer[String]("topic", new SimpleStringSchema(), properties))


////////////////////////////////////////////////////////////////
//正则匹配topic
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "test");

FlinkKafkaConsumer<String> myConsumer = new FlinkKafkaConsumer<>(
    java.util.regex.Pattern.compile("test-topic-[0-9]"),
    new SimpleStringSchema(),
    properties);

DataStream<String> stream = env.addSource(myConsumer);
```

**DeserializationSchema** 

Flink Kafka Consumer 需要知道如何将 Kafka 中的二进制数据转换为 Java 或者 Scala 对象。`KafkaDeserializationSchema` 允许用户指定这样的 schema，每条 Kafka 中的消息会调用 `T deserialize(ConsumerRecord<byte[], byte[]> record)` 反序列化。

为了方便使用，Flink 提供了以下几种 schemas：

1. `TypeInformationSerializationSchema`（和 `TypeInformationKeyValueSerializationSchema`) 基于 Flink 的 `TypeInformation` 创建 `schema`。 如果该数据的读和写都发生在 Flink 中，那么这将是非常有用的。此 schema 是其他通用序列化方法的高性能 Flink 替代方案。
2. `JsonDeserializationSchema`（和 `JSONKeyValueDeserializationSchema`）将序列化的 JSON 转化为 ObjectNode 对象，可以使用 `objectNode.get("field").as(Int/String/...)()` 来访问某个字段。 KeyValue objectNode 包含一个含所有字段的 key 和 values 字段，以及一个可选的"metadata"字段，可以访问到消息的 offset、partition、topic 等信息。
3. `AvroDeserializationSchema` 使用静态提供的 schema 读取 Avro 格式的序列化数据。 它能够从 Avro 生成的类（`AvroDeserializationSchema.forSpecific(...)`）中推断出 schema，或者可以与 `GenericRecords` 一起使用手动提供的 schema（用 `AvroDeserializationSchema.forGeneric(...)`）。此反序列化 schema 要求序列化记录不能包含嵌入式架构！
   - 此模式还有一个版本，可以在 Confluent Schema Registry中查找编写器的 schema（用于编写记录的 schema）。
   - 使用这些反序列化 schema 记录将读取从 schema 注册表检索到的 schema 转换为静态提供的 schema（或者通过 `ConfluentRegistryAvroDeserializationSchema.forGeneric(...)` 或 `ConfluentRegistryAvroDeserializationSchema.forSpecific(...)`）。

**配置开始消费位置**

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

val myConsumer = new FlinkKafkaConsumer[String](...)
myConsumer.setStartFromEarliest()      // 尽可能从最早的记录开始
myConsumer.setStartFromLatest()        // 从最新的记录开始
myConsumer.setStartFromTimestamp(...)  // 从指定的时间开始（毫秒）
myConsumer.setStartFromGroupOffsets()  // 默认的方法

val stream = env.addSource(myConsumer)

////////////////////////////////////////////////
//每个分区指定开始消费位置
val specificStartOffsets = new java.util.HashMap[KafkaTopicPartition, java.lang.Long]()
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 0), 23L)
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 1), 31L)
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 2), 43L)

myConsumer.setStartFromSpecificOffsets(specificStartOffsets)
```

- `setStartFromGroupOffsets`（默认方法）：从 Kafka brokers 中的 consumer 组（consumer 属性中的 `group.id` 设置）提交的偏移量中开始读取分区。 如果找不到分区的偏移量，那么将会使用配置中的 `auto.offset.reset` 设置。
- `setStartFromEarliest()` 或者 `setStartFromLatest()`：从最早或者最新的记录开始消费，在这些模式下，Kafka 中的 committed offset 将被忽略，不会用作起始位置。
- `setStartFromTimestamp(long)`：从指定的时间戳开始。对于每个分区，其时间戳大于或等于指定时间戳的记录将用作起始位置。如果一个分区的最新记录早于指定的时间戳，则只从最新记录读取该分区数据。在这种模式下，Kafka 中的已提交 offset 将被忽略，不会用作起始位置。

### sink

```scala
val properties = new Properties
properties.setProperty("bootstrap.servers", "localhost:9092")

val myProducer = new FlinkKafkaProducer[String](
        "my-topic",               // 目标 topic
        new SimpleStringSchema(), // 序列化 schema
        properties,               // producer 配置
        FlinkKafkaProducer.Semantic.EXACTLY_ONCE) // 容错

stream.addSink(myProducer)
```

**SerializationSchema** 

Flink Kafka Producer 需要知道如何将 Java/Scala 对象转化为二进制数据。

`KafkaSerializationSchema` 允许用户指定这样的 schema。它会为每个记录调用 `ProducerRecord<byte[], byte[]> serialize(T element, @Nullable Long timestamp)` 方法，产生一个写入到 Kafka 的 `ProducerRecord`。

**Kafka Producer 和容错**

启用 Flink 的 checkpointing 后，`FlinkKafkaProducer` 可以提供精确一次的语义保证。

除了启用 Flink 的 checkpointing，你也可以通过将适当的 `semantic` 参数传递给 `FlinkKafkaProducer` 来选择三种不同的操作模式：

- `Semantic.NONE`：Flink 不会有任何语义的保证，产生的记录可能会丢失或重复。
- `Semantic.AT_LEAST_ONCE`（默认设置）：可以保证不会丢失任何记录（但是记录可能会重复）
- `Semantic.EXACTLY_ONCE`：使用 Kafka 事务提供精确一次语义。无论何时，在使用事务写入 Kafka 时，都要记得为所有消费 Kafka 消息的应用程序设置所需的 `isolation.level`（`read_committed` 或 `read_uncommitted` - 后者是默认值）。

##### 注意事项

`Semantic.EXACTLY_ONCE` 模式依赖于事务提交的能力。事务提交发生于触发 checkpoint 之前，以及从 checkpoint 恢复之后。如果从 Flink 应用程序崩溃到完全重启的时间超过了 Kafka 的事务超时时间，那么将会有数据丢失（Kafka 会自动丢弃超出超时时间的事务）。考虑到这一点，请根据预期的宕机时间来合理地配置事务超时时间。

默认情况下，Kafka broker 将 `transaction.max.timeout.ms` 设置为 15 分钟。此属性不允许为大于其值的 producer 设置事务超时时间。 默认情况下，`FlinkKafkaProducer` 将 producer config 中的 `transaction.timeout.ms` 属性设置为 1 小时，因此在使用 `Semantic.EXACTLY_ONCE` 模式之前应该增加 `transaction.max.timeout.ms` 的值。

在 `KafkaConsumer` 的 `read_committed` 模式中，任何未结束（既未中止也未完成）的事务将阻塞来自给定 Kafka topic 的未结束事务之后的所有读取数据。 换句话说，在遵循如下一系列事件之后：

1. 用户启动了 `transaction1` 并使用它写了一些记录
2. 用户启动了 `transaction2` 并使用它编写了一些其他记录
3. 用户提交了 `transaction2`

即使 `transaction2` 中的记录已提交，在提交或中止 `transaction1` 之前，消费者也不会看到这些记录。这有 2 层含义：

- 首先，在 Flink 应用程序的正常工作期间，用户可以预料 Kafka 主题中生成的记录的可见性会延迟，相当于已完成 checkpoint 之间的平均时间。
- 其次，在 Flink 应用程序失败的情况下，此应用程序正在写入的供消费者读取的主题将被阻塞，直到应用程序重新启动或配置的事务超时时间过去后，才恢复正常。此标注仅适用于有多个 agent 或者应用程序写入同一 Kafka 主题的情况。

**注意**：`Semantic.EXACTLY_ONCE` 模式为每个 `FlinkKafkaProducer` 实例使用固定大小的 KafkaProducer 池。每个 checkpoint 使用其中一个 producer。如果并发 checkpoint 的数量超过池的大小，`FlinkKafkaProducer` 将抛出异常，并导致整个应用程序失败。请合理地配置最大池大小和最大并发 checkpoint 数量。

**注意**：`Semantic.EXACTLY_ONCE` 会尽一切可能不留下任何逗留的事务，否则会阻塞其他消费者从这个 Kafka topic 中读取数据。但是，如果 Flink 应用程序在第一次 checkpoint 之前就失败了，那么在重新启动此类应用程序后，系统中不会有先前池大小（pool size）相关的信息。因此，在第一次 checkpoint 完成前对 Flink 应用程序进行缩容，且并发数缩容倍数大于安全系数 `FlinkKafkaProducer.SAFE_SCALE_DOWN_FACTOR` 的值的话，是不安全的。

## Others参见官方文档

flink->documentation->connectors

jdbc,es,file(hadoop),redis.....

