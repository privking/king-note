# Kafka基础

Kafka是一个`分布式`的基于`发布/订阅模式`的`消息队列`（Message Queue），主要应用于大数据实时处理领域

- 解耦
  - 组件之间互不影响
-  异步 
  - 对于不需要及时处理的消息，可以使用异步处理机制
- 流量削峰
  - 解决生产和处理消息不一致的问题
-  可恢复性
  - 在消费者挂了后，重启能够继续上次的位置消费
- 灵活性
  - 可以动态修改机器数量

![image-20201025112850192](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025112850192-1603596537-641d24.png)

 

## 消息队列两种模式

### 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）

消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。

消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有`一个消费者`可以消费

![image-20201025114252684](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025114252684-1603597372-402f21.png)

### 发布/订阅模式（一对多，消费者消费数据之后不会清除消息）

消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

![image-20201025114308652](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025114308652-1603597388-d61cf1.png)



## kafka基础架构

### Broker

一台kafka服务器就是一个broker。一个集群由多个broker组成

### Topic

Topic 就是数据主题，kafka建议根据业务系统将不同的数据存放在不同的topic中！Kafka中的Topics总是多订阅者模式，一个topic可以拥有一个或者多个消费者来订阅它的数据。一个大的Topic可以分布式存储在多个kafka broker中！

### Partition

- 每个topic可以有多个分区，通过分区的设计，topic可以不断进行扩展！即一个Topic的多个分区分布式存储在多个broker!
- 此外通过分区还可以让一个topic被多个consumer进行消费！以达到并行处理！
- kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。
- 物理层面分区 存储在不同目录  保存在logDir   以topicname-0,topicname-1命名

### Offset

- 数据会按照时间顺序被不断第追加到分区的一个结构化的commit log中！每个分区中存储的记录都是有序的，且顺序不可变！
- 这个顺序是通过一个称之为offset的id来唯一标识！因此也可以认为offset是有序且不可变的！ 
- 在每一个消费者端，会唯一保存的元数据是offset（偏移量）,即消费在log中的位置.`偏移量由消费者所控制`。通常在读取记录后，消费者会以线性的方式增加偏移量，但是实际上，由于这个位置由消费者控制，所以消费者可以采用任何顺序来消费记录。例如，一个消费者可以重置到一个旧的偏移量，从而重新处理过去的数据；也可以跳过最近的记录，从"现在"开始消费。
- 这些细节说明Kafka 消费者是非常廉价的—消费者的增加和减少，对集群或者其他消费者没有多大的影响。比如，你可以使用命令行工具，对一些topic内容执行 tail操作，并不会影响已存在的消费者消费数据。

![image-20201025161432018](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025161432018-1603613672-830a59.png)

### 持久化

Kafka 集群保留所有发布的记录—无论他们是否已被消费—并通过一个可配置的参数——保留期限来控制。举个例子， 如果保留策略设置为2天，一条记录发布后两天内，可以随时被消费，两天过后这条记录会被清除并释放磁盘空间。

Kafka的`性能和数据大小无关`，所以长时间存储数据没有什么问题

### 副本机制

- 日志的分区partition （分布）在Kafka集群的服务器上。每个服务器在处理数据和请求时，共享这些分区。每一个分区都会在已配置的服务器上进行备份，确保容错性。
- 每个分区都有一台 server 作为 “leader”，零台或者多台server作为 follwers 。leader server 处理一切对 partition （分区）的读写请求，而follwers只需`被动的同步leader上的数据`。当leader宕机了，followers 中的一台服务器会自动成为新的 leader。通过这种机制，既可以保证数据有多个副本，也实现了一个高可用的机制！
- 基于安全考虑，每个分区的Leader和follower一般会错在在不同的broker

### Producer

消息生产者，就是向kafka broker发消息的客户端。生产者负责将记录分配到topic的指定 partition（分区）中

### Consumer

消息消费者，向kafka broker取消息的客户端。每个消费者都要维护自己读取数据的offset。低版本0.9之前将offset保存在Zookeeper中，0.9及之后保存在Kafka的“**__consumer_offsets**”主题中。

### Consumer Group

- 每个消费者都会使用一个消费组名称来进行标识。同一个组中的不同的消费者实例，可以`分布在多个进程或多个机器上`！
- 如果所有的消费者实例在同一消费组中，消息记录会`负载平衡到每一个消费者实例`（单播）。即每个消费者可以`同时读取一个topic的不同分区`！
- 如果所有的消费者实例在不同的消费组中，每条消息记录会广播到所有的消费者进程(广播)。
- 如果需要实现广播，只要每个consumer有一个独立的组就可以了。要实现单播只要所有的consumer在同一个组。
- 一个topic可以有多个consumer group。topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partion只会把消息发给该CG中的一个consumer。

## 文件储存

```
# The maximum size of a log segment file. When this size is reached a new log segment will be created.
# 数据文件片段大小，超过该大小创建新文件
log.segment.bytes=1073741824
```

![image-20201025174018566](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025174018566-1603618818-c4d670.png)

生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了**分片**和**索引**机制，将每个partition分为多个segment

每个segment对应两个文件（**新版本增加了TimeIndex索引，映射时间戳和相对offset, 时间戳和相对offset作为entry,供占用12字节，时间戳占用8字节，相对offset占用4字节，这个索引也是稀疏索引，没有保存全部的消息的entry**）——“.index”文件和“.log”文件。分别表示为segment索引文件和数据文件（引入索引文件的目的就是便于利用**二分查找**快速定位message位置）。这两个文件的命令规则为：**partition全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的《offset》值**，数值大小为64位，19位数字字符长度，没有数字用0填充。

这些文件位于一个文件夹下（partition目录），该文件夹的命名规则为：topic名称+分区序号。例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2。

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

![image](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/c415ed42-1603619660-e7c11c.png)

- 第一步查找segment file 上述图2为例，其中00000000000000000000.index表示最开始的文件，起始偏移量(offset)为0.第二个文件00000000000000368769.index的消息量起始偏移量为368770 = 368769 + 1.同样，第三个文件00000000000000737337.index的起始偏移量为737338=737337 + 1，其他后续文件依次类推，以起始偏移量命名并排序这些文件，只要根据offset **二分查找**文件列表，就可以快速定位到具体文件。 当offset=368776时定位到00000000000000368769.index|log
- 第二步通过segment file查找message 通过第一步定位到segment file，当offset=368776时，依次定位到00000000000000368769.index的元数据物理位置和00000000000000368769.log的物理偏移地址，然后再通过00000000000000368769.log顺序查找直到offset=368776为止。

## 数据可靠性

为保证producer发送的数据，能可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement确认收到），如果producer收到ack，就会进行下一轮的发送，否则重新发送数据。

![image-20201025180231440](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025180231440-1603620151-7fbb5d.png)



### 副本数据同步策略

| **方案**                        | **优点**                                           | **缺点**                                                     |
| ------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **半数以上完成同步，就发送ack** | 延迟低                                             | 选举新的leader时，容忍n台节点的故障，需要2n+1个副本 (数据冗余大) |
| **全部完成同步，才发送ack**     | 选举新的leader时，容忍n台节点的故障，需要n+1个副本 | 延迟高                                                       |

### ISR ！！！！！！

- 采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与leader进行同步，那leader就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？
- Leader维护了一个动态的in-sync replica set (ISR)，意为**和leader保持同步的follower集合**。当ISR中的follower完成数据的同步之后，leader就会给follower发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由**replica.lag.time.max.ms**参数设定。（0.9版本参数条件还有最大相差条数。已经移除）Leader发生故障之后，就会从ISR中选举新的leader。



### ack应答机制 

- 对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等ISR中的follower全部接收成功。
- 所以Kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。
- **acks**：
  - 0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没有写入磁盘就已经返回，当broker故障时有可能**丢失数据**；
  - 1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果在isr中follower同步成功之前leader故障，那么将会**丢失数据**；
  - -1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成**数据重复**。

### 故障处理细节

![image-20201025182606071](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025182606071-1603621566-8ac3d1.png)

- LEO：指的是每个副本最大的offset；
- HW：指的是消费者能见到的最大的offset，ISR队列中最小的LEO。
- follower故障
  - follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。
- leader故障
  - leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。

**注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。**



### Exactly Once 

将服务器ack设置成-1 ，可以保证producer和broker之间不会丢数据，及**At Least Once**语义

将服务器ack设置为0，可以保证producer最多发送一次消息，及**At Most Once**语义

相对于一些重要的信息，比如交易数据，下游消费者要求既不能重复消费也不能丢失，及**Exactly Once**语义

`At Least Once+幂等性=Exactly Once`

启用幂等性，只需要将Producer的参数`enable.idompotence`设置成`true` ,Producer在初始化的时候会被分配一个PID(producerId)

发往同一Partition的消息会附带 Sequence Number ，而Broker会对`<PID,Partition,SeqNumber>`做缓存，当具有相同主键的消息只会被持久化一条。

但是重启后PID就会变化，Partition也可能会变化，所以`Exactly Once`不能保证跨分区，跨会话

## 消费者消费方式

- consumer采用pull（拉）模式从broker中读取数据。
- push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。
- pull模式不足之处是，如果kafka没有数据，消费者可能会`陷入循环`中，一直返回空数据。针对这一点，Kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。

## 分区分配策略

- 一个consumer group中有多个consumer，一个 topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费。

- Kafka有两种分配策略，一是RoundRobin，一是Range。

- 重新分区分配

  - 同一个 Consumer Group 内新增消费者
  - 消费者离开当前所属的Consumer Group，包括shuts down 或 crashes
  - 订阅的主题新增分区

  ![image-20201025231836586](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025231836586-1603639116-2ec7bc.png)

![image-20201025231912815](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025231912815-1603639152-0178e5.png)

![image-20201025231940665](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201025231940665-1603639180-99a1ce.png)

## Kafka事务

Kafka在0.11版本中除了引入了Exactly Once语义，还引入了事务特性。**Kafka事务特性是指一系列的生产者生产消息和消费者提交偏移量的操作在一个事务中，或者说是一个原子操作，生产消息和提交偏移量同时成功或者失败**

Kafka中的事务特性主要用于以下两种场景：

- **生产者发送多条消息可以封装在一个事务中，形成一个原子操作。**多条消息要么都发送成功，要么都发送失败。
- **read-process-write模式：将消息消费和生产封装在一个事务中，形成一个原子操作。**在一个流式处理的应用中，常常一个服务需要从上游接收消息，然后经过处理后送达到下游，这就对应着消息的消费和生成。
- 引入TransactionId：不同生产实例使用同一个TransactionId表示是同一个事务，可以`跨Session的数据幂等发送`。当具有相同Transaction ID的新的Producer实例被创建且工作时，旧的且拥有相同Transaction ID的Producer将不再工作，避免事务僵死。

> 当事务中仅仅存在Consumer消费消息的操作时，它和Consumer手动提交Offset并没有区别。因此单纯的消费消息并不是Kafka引入事务机制的原因，单纯的消费消息也没有必要存在于一个事务中。

```cpp
    /**
     * 初始化事务
     */
    public void initTransactions();
 
    /**
     * 开启事务
     */
    public void beginTransaction() throws ProducerFencedException ;
 
    /**
     * 在事务内提交已经消费的偏移量
     */
    public void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets, 
                                         String consumerGroupId) throws ProducerFencedException ;
 
    /**
     * 提交事务
     */
    public void commitTransaction() throws ProducerFencedException;
 
    /**
     * 丢弃事务
     */
    public void abortTransaction() throws ProducerFencedException ;
```

```bash
KafkaProducer producer = createKafkaProducer(
  "bootstrap.servers", "localhost:9092",
  "transactional.id”, “my-transactional-id");

producer.initTransactions();
producer.beginTransaction();
producer.send("outputTopic", "message1");
producer.send("outputTopic", "message2");
producer.commitTransaction();
```



## 高效读写数据

### 顺序写磁盘

Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到600M/s，而随机写只有100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

### 零拷贝技术

