# Environment&Source

## Envrionment

### getExecutionEnvironment

创建一个执行环境，表示当前执行程序的上下文。 **如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境**，也就是说，getExecutionEnvironment会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式。

```scala
//并行度默认为flink-conf.yaml中的配置
val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

val env = StreamExecutionEnvironment.getExecutionEnvironment
```

### createLocalEnvironment

返回本地执行环境，需要在调用时指定默认的并行度。

```scala
val env = StreamExecutionEnvironment.createLocalEnvironment(1)
```

### createRemoteEnvironment

返回集群执行环境，将Jar提交到远程服务器。需要在调用时指定JobManager的IP和端口号，并指定要在集群中运行的Jar包

```scala
val env = ExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123,"YOURPATH//wordcount.jar")
```

## Source

### 从集合

```scala
// 定义样例类，传感器id，时间戳，温度
case class SensorReading(id: String, timestamp: Long, temperature: Double)

object Sensor {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream1 = env
      .fromCollection(List(
        SensorReading("sensor_1", 1547718199, 35.8),
        SensorReading("sensor_6", 1547718201, 15.4),
        SensorReading("sensor_7", 1547718202, 6.7),
        SensorReading("sensor_10", 1547718205, 38.1)
      ))

    stream1.print("stream1:").setParallelism(1)

    env.execute()
  }
}
```

### 从文件

```scala
val stream2 = env.readTextFile("YOUR_FILE_PATH")
```

### Kafka

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
<!---0.11  ->kafka版本-->
<!---2.11  ->scala版本-->

```

```scala
val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
properties.setProperty("group.id", "consumer-group")
properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("auto.offset.reset", "latest")

val stream3 = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

```

### 自定义source

```scala
val stream4 = env.addSource( new MySensorSource() )


class MySensorSource extends SourceFunction[SensorReading]{

// flag: 表示数据源是否还在正常运行
var running: Boolean = true

override def cancel(): Unit = {
	running = false
}

override def run(ctx: SourceFunction.SourceContext[SensorReading]): Unit = {
    // 初始化一个随机数发生器
    val rand = new Random()

    var curTemp = 1.to(10).map(
        //高斯随机
   		i => ( "sensor_" + i, 65 + rand.nextGaussian() * 20 )
    )

    while(running){
        // 更新温度值
        curTemp = curTemp.map(
        	t => (t._1, t._2 + rand.nextGaussian() )
        )
        // 获取当前时间戳
        val curTime = System.currentTimeMillis()

        curTemp.foreach(
            //采集数据
        	t => ctx.collect(SensorReading(t._1, curTime, t._2))
        )
        Thread.sleep(100)
        }
    }
}

```

