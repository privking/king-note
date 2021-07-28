# sTransform

## map

```scala
val streamMap = stream.map { x => x * 2 }
```

## flatMap

```scala
val streamFlatMap = stream.flatMap{

  x => x.split(" ")

}
```

## filter

```scala
val streamFilter = stream.filter{

  x => x == 1

}
```

## KeyBy

**DataStream** **→** **KeyedStream**：逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同key的元素，在内部以hash的形式实现的。

```scala
//按照下标
def keyBy(fields: Int*): KeyedStream[T, JavaTuple] 
//按照字段名（比如样例类） 返回JavaTuple
def keyBy(firstField: String, otherFields: String*): KeyedStream[T, JavaTuple] 
//传递函数
def keyBy[K: TypeInformation](fun: T => K): KeyedStream[T, K]
//KeySelector -->@FunctionalInterface
def keyBy[K: TypeInformation](fun: KeySelector[T, K]): KeyedStream[T, K] 
```

## 滚动聚合算子

这些算子可以针对**KeyedStream**的每一个支流做聚合。

### sum

```scala
//下标
def sum(position: Int): DataStream[T]
//字段名
def sum(field: String): DataStream[T] 

```

### min&minBy

```scala
//比如泛型是一个样例类，只有求最小值的字段是最小值，其他字段是第一个输入的原始值，不会改变（只有最小值字段替换）
//下标
def min(position: Int): DataStream[T] 
//字段名
def min(field: String): DataStream[T] 

//比如泛型是一个样例类，求最小值的字段是最小值，其他字段是最小值对应的值，（整个对象替换）
//下标
def minBy(position: Int): DataStream[T]
//字段名
def minBy(field: String): DataStream[T] 
```

### max&maxBy

```scala
//下标
def max(position: Int): DataStream[T]
//字段名
def max(field: String): DataStream[T]
//下标
def maxBy(position: Int): DataStream[T] 
//字段名
def maxBy(field: String): DataStream[T]
```

### reduce

```scala
//函数
def reduce(fun: (T, T) => T): DataStream[T] 
//@FunctionalInterface
def reduce(reducer: ReduceFunction[T]): DataStream[T] 
```

 

## split

**DataStream** **→** **SplitStream**：根据某些特征把一个DataStream拆分成两个或者多个DataStream。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clip_image002-1627486859-76e321.jpg)

## select

**SplitStream**→**DataStream**：从一个SplitStream中获取一个或者多个DataStream。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clip_image002-1627486867-a09945.jpg)

```scala
    val splitStream: SplitStream[String] = stream
      .split( data => {
        if (data.length>5)
          Seq("high")
        else
          Seq("low")
      } )

    val high: DataStream[String] = splitStream.select("high")
    val low: DataStream[String] = splitStream.select("low")
    val all: DataStream[String] = splitStream.select("high", "low")
```

## Connect

**DataStream,DataStream** **→** **ConnectedStreams**：连接两个保持他们类型的数据流，两个数据流被Connect之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clip_image002-1627487072-06ac45.jpg)

##  CoMap,CoFlatMap

**ConnectedStreams → DataStream**：作用于ConnectedStreams上，功能与map和flatMap一样，对ConnectedStreams中的每一个Stream分别进行map和flatMap处理。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clip_image002-1627487100-55c67e.jpg)

```scala
val warning = high.map( sensorData => (sensorData.id, sensorData.temperature) )
val connected = warning.connect(low)

val coMap = connected.map(
    warningData => (warningData._1, warningData._2, "warning"),
    lowData => (lowData.id, "healthy")
)
```

## union

**DataStream** **→** **DataStream**：对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clip_image002-1627487209-342a94.jpg)

```scala
val unionStream: DataStream[StartUpLog] = appStoreStream.union(otherStream)
unionStream.print("union:::")

```

**Connect与Union 区别：**

- Union之前两个流的类型必须是一样，Connect可以不一样，在之后的coMap中再去调整成为一样的。
- Connect只能操作两个流，Union可以操作多个。

