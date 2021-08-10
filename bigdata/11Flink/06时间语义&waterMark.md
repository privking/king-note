# 时间语义&WaterMark

## 时间语义

**Processing time**

执行相应操作的机器的系统时间。

Processing time是最简单的时间概念，不需要流和机器之间的协调。它提供了最好的性能和最低的延迟。

**IngestionTime**

进入flink的时间

**Event time**

Event time是每个事件在其产生设备上发生的时间。这个时间通常在记录进入Flink之前嵌入到记录中，并且**可以从每个记录中提取事件时间戳**。在事件时间中，**时间的进展取决于数据，而不是任何挂钟**。事件时间程序必须指定如何生成事件时间水印(Event time Watermarks)，**这是一种以事件时间表示进展程度的机制**。

**解决数据在处理时是一种乱序状态的方式**

![image-20210809204822436](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210809204822436-1628513302-c18c35.png)

## WaterMark

水印就是一个时间戳

水印对于乱序流至关重要。一般来说，水印是一种声明，表示**到流中的那个点，在某个时间戳之前的所有事件都应该已经到达**。相当于表示现在的时间为，**当前事件时间-最大乱序程度**

![image-20210809205622381](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210809205622381-1628513782-4301b2.png)

**在并行流中的WaterMark**

水印一般是在sourceFunction,或者在sourceFunction的紧挨后面，每一个子任务的都会独立的生成自己的水印

当进行shuffle时，水印以最小值为准

![image-20210809210142792](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210809210142792-1628514102-b5a1ec.png)

### 设置watermark

```scala
object WindowTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    env.getConfig.setAutoWatermarkInterval(100L)

    val inputStream = env.socketTextStream("hadoop102", 7777)
    val dataStream: DataStream[SensorReading] = inputStream
      .map( data => {
        val dataArray = data.split(",")
        SensorReading( dataArray(0), dataArray(1).toLong, dataArray(2).toDouble )
      } )
//    .assignAscendingTimestamps(_.timestamp * 1000L)
//    .assignTimestampsAndWatermarks( new MyWMAssigner(1000L) )
      //AssignerWithPeriodicWatermarks
      .assignTimestampsAndWatermarks( new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(1)) {
      override def extractTimestamp(element: SensorReading): Long = element.timestamp * 1000L
    } ).setParallelism(2)

    val resultStream = dataStream
      .keyBy("id")
//    .window( EventTimeSessionWindows.withGap(Time.minutes(1)) )    // 会话窗口
//    .window( TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8)) )
      .timeWindow(Time.seconds(15), Time.seconds(5))
      .allowedLateness(Time.minutes(1))
      .sideOutputLateData(new OutputTag[SensorReading]("late"))
//    .reduce( new MyReduce() )
      .apply( new MyWindowFun() )

    dataStream.print("data2")
    resultStream.getSideOutput(new OutputTag[SensorReading]("late"))
    resultStream.print("result2")
    env.execute("window test")
  }
}
```

```scala
//Periodic Watermark
//周期性的（一定时间间隔或者达到一定的记录条数）产生一个 Watermark。
//默认200毫秒 env.getConfig.setAutoWatermarkInterval(200)
def assignTimestampsAndWatermarks(assigner: AssignerWithPeriodicWatermarks[T]): DataStream[T] 
//Punctuated Watermark
//数据流中每一个递增的 EventTime 都会产生一个 Watermark。
def assignTimestampsAndWatermarks(assigner: AssignerWithPunctuatedWatermarks[T]): DataStream[T] 
```

```scala
// 自定义一个周期性生成watermark的Assigner
class MyWMAssigner(lateness: Long) extends AssignerWithPeriodicWatermarks[SensorReading]{
  // 需要两个关键参数，延迟时间，和当前所有数据中最大的时间戳
//  val lateness: Long = 1000L
  var maxTs: Long = Long.MinValue + lateness

  override def getCurrentWatermark: Watermark =
    new Watermark(maxTs - lateness)

  override def extractTimestamp(element: SensorReading, previousElementTimestamp: Long): Long = {
    maxTs = maxTs.max(element.timestamp * 1000L)
    element.timestamp * 1000L
  }
}

// 自定义一个断点式生成watermark的Assigner
class MyWMAssigner2 extends AssignerWithPunctuatedWatermarks[SensorReading]{
  val lateness: Long = 1000L
   //每来一个元素都会调用该方法
  override def checkAndGetNextWatermark(lastElement: SensorReading, extractedTimestamp: Long): Watermark = {
    if( lastElement.id == "sensor_1" ){
      new Watermark(extractedTimestamp - lateness)
    } else
      null
  }

  override def extractTimestamp(element: SensorReading, previousElementTimestamp: Long): Long =
    element.timestamp * 1000L
}
```

### 在10.11中新增了WatermarkStrategy

```scala
stream.flatMap(_.split(" "))
  .map((_, 1))
  .assignTimestampsAndWatermarks(WatermarkStrategy
    .forBoundedOutOfOrderness[(String, Int)](Duration.ofSeconds(20))
    .withIdleness(Duration.ofMinutes(1)))

  .keyBy(_._1)
  .window(TumblingEventTimeWindows.of(Time.hours(1)))
  .sum(2)
  .print()

////////////////////////////////
def assignTimestampsAndWatermarks(watermarkStrategy: WatermarkStrategy[T]): DataStream[T]
```

WatermarkStrategy 这里面提供了很多静态的方法和带有缺省实现的方法，只有一个方法是非default和没有缺省实现的，就是下面的这个方法。

```java
WatermarkGenerator<T> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context);
//可以使用以下方法代替
 static <T> WatermarkStrategy<T> forGenerator(WatermarkGeneratorSupplier<T> generatorSupplier) {
        return generatorSupplier::createWatermarkGenerator;
 }
```

```scala
@Public
public interface WatermarkGenerator<T> {

 /**
  * Called for every event, allows the watermark generator to examine and remember the
  * event timestamps, or to emit a watermark based on the event itself.
  
  onEvent ：每个元素都会调用这个方法，如果我们想依赖每个元素生成一个水印，然后发射到下游(可选，就是看是否用output来收集水印)，我们可以实现这个方法.
  
  */
 void onEvent(T event, long eventTimestamp, WatermarkOutput output);

 /**
  * Called periodically, and might emit a new watermark, or not.
  *
  * <p>The interval in which this method is called and Watermarks are generated
  * depends on {@link ExecutionConfig#getAutoWatermarkInterval()}.
  
  onPeriodicEmit : 如果数据量比较大的时候，我们每条数据都生成一个水印的话，会影响性能，所以这里还有一个周期性生成水印的方法。这个水印的生成周期可以这样设置：env.getConfig().setAutoWatermarkInterval(5000L);
  
  */
 void onPeriodicEmit(WatermarkOutput output);
}
```

#### 内置生成策略

**固定延迟**

```scala
DataStream dataStream = ...... ;
dataStream.assignTimestampsAndWatermarks(WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(3)));
```

**单调递增生成水印**

```scala
//相当于上述的延迟策略去掉了延迟时间，以event中的时间戳充当了水印
DataStream dataStream = ...... ;
dataStream.assignTimestampsAndWatermarks(WatermarkStrategy.forMonotonousTimestamps());
```

**自定义时间抽取**

```scala
DataStream dataStream = ...... ;
dataStream.assignTimestampsAndWatermarks(
    WatermarkStrategy
      .<Tuple2<String,Long>>forBoundedOutOfOrderness(Duration.ofSeconds(5))
     //kafka等自带时间戳的可以不用重新声明
      .withTimestampAssigner((event, timestamp)->event.f1));
```

**处理空闲数据源**

在某些情况下，由于数据产生的比较少，导致一段时间内没有数据产生，进而就没有水印的生成，导致下游依赖水印的一些操作就会出现问题，比如某一个算子的上游有多个算子，这种情况下，水印是取其上游两个算子的较小值，如果上游某一个算子因为缺少数据迟迟没有生成水印，就会出现eventtime倾斜问题，导致下游没法触发计算。

所以filnk通过WatermarkStrategy.withIdleness()方法允许用户在配置的时间内（即超时时间内）没有记录到达时将一个流标记为空闲。这样就意味着下游的数据不需要等待水印的到来。

当下次有水印生成并发射到下游的时候，这个数据流重新变成活跃状态。

```scala
WatermarkStrategy
        .<Tuple2<Long, String>>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withIdleness(Duration.ofMinutes(1));
```

