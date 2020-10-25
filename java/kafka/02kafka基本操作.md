# Kafka基本操作

## kafka安装

### server.properties

```properties
#broker的全局唯一编号，不能重复:
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘IO的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka日志存放的路径,数据也是以日志方式存储
log.dirs=/usr/local/soft/kafka/kafka_2.12-2.6.0/data
#topic在当前broker上的分区个数
num.partitions=1
#用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment文件保留的最长时间，超时将被删除 168小时-》默认一周
log.retention.hours=168
#配置连接Zookeeper集群地址
zookeeper.connect=node1:2181,node2:2181,node3:2181

```

### 启动kafka server

1. 启动配置好的zookeeper

2. `bin/kafka-server-start.sh -deamon config/server.properties`
3. 逐台启动 或写脚本
4. 停止`bin/kafka-server-stop.sh`

## Kafka命令

`--zookeeper  zkip:zkport` 过时 替换为 `--bootstrap-server kafkaip:kafkaport`

### 查看所有的topics

```sh
./kafka-topics.sh --list --zookeeper  node1:2181

./kafka-topics.sh --list --bootstrap-server node1:9092
```

### 创建topic 

```sh
./kafka-topics.sh --create --topic hello --partitions 2 --replication-factor 2 --zookeeper  node1:2181

./kafka-topics.sh --create --topic hello --partitions 2 --replication-factor 2 --bootstrap-server node1:9092

# 分区数数量无上限
# 副本数量不能超过集群的总数
```

### 删除topic

```sh
./kafka-topics.sh --delete --topic hello  --zookeeper node1:2181

./kafka-topics.sh --delete --topic hello  --bootstrap-server node1:9092
```

### 查看topic详情

```sh
./kafka-topics.sh --describe hello --zookeeper node1:2181

./kafka-topics.sh --describe --topic hello --bootstrap-server node1:9092

# Topic: hello	PartitionCount: 2	ReplicationFactor: 2	Configs: 
# 	Topic: hello	Partition: 0	Leader: 1	Replicas: 1,3	Isr: 1,3
# 	Topic: hello	Partition: 1	Leader: 2	Replicas: 2,1	Isr: 2,1
```

### 修改topic

```sh
./kafka-topics.sh  --alter --topic hello --partitions 6 --zookeeper node1:2181

./kafka-topics.sh  --alter --topic hello --partitions 6 --bootstrap-server node1:9092
```

### console-producer

```sh
./kafka-console-producer.sh  --topic hello --broker-list node1:9092
```

### console-consumer

```sh
./kafka-console-consumer.sh --topic hello --bootstrap-server node1:9092

--from-beginning 设置offset为起始位置 
```

