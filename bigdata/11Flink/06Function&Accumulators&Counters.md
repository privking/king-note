# Function&Accumulator&Counter

## Function

MapFunction,FilterFunction,FlatMapFunction.......

```scala
stream.map(new MyMapFuntion)

class MyMapFuntion extends MapFunction[String,(String,Int)]{
  override def map(value: String): (String, Int) = ???
}
```

RichMapFunction,RichFilterFunction........

```scala
data.map(new MyRichMapFunction());


class MyRichMapFunction extends RichMapFunction[String, Int] {
  //map
  def map(in: String):Int = { in.toInt }

  //开始时调用
  override def open(parameters: Configuration): Unit = super.open(parameters)

  override def setRuntimeContext(t: RuntimeContext): Unit = super.setRuntimeContext(t)
  //获取context
  override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

  override def getIterationRuntimeContext: IterationRuntimeContext = super.getIterationRuntimeContext
  
  //结束时调用
  override def close(): Unit = super.close()
};
```

## Accumulator&Counter

累加器是具有**加法运算**和**最终累加结果**的一种简单结构，可在作业结束后使用。

最简单的累加器就是**计数器**: 你可以使用 `Accumulator.add(V value)` 方法将其递增。在作业结束时，Flink 会汇总（合并）所有部分的结果并将其发送给客户端。 在调试过程中或在你想快速了解有关数据更多信息时,累加器作用很大。

IntCounter,LongCounter,DoubleCounter

首先，在需要使用累加器的用户自定义的转换 function 中创建一个累加器对象（此处是计数器）。

```java
private IntCounter numLines = new IntCounter();
```

其次，你必须在 *rich* function 的 `open()` 方法中注册累加器对象。也可以在此处定义名称。

```java
getRuntimeContext().addAccumulator("num-lines", this.numLines);
```

现在你可以在操作 function 中的任何位置（包括 `open()` 和 `close()` 方法中）使用累加器。

```java
this.numLines.add(1);
```

最终整体结果会存储在由执行环境的 `execute()` 方法返回的 `JobExecutionResult` 对象中（当前只有等待作业完成后执行才起作用）。

```java
myJobExecutionResult.getAccumulatorResult("num-lines")
```

**单个作业的所有累加器共享一个命名空间。因此你可以在不同的操作 function 里面使用同一个累加器。Flink 会在内部将所有具有相同名称的累加器合并起来。**

**当前累加器的结果只有在整个作业结束后才可用**

