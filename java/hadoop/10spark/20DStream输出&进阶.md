# DStream输出&进阶

## 输出

与RDD中的惰性求值类似，如果一个DStream及其派生出的DStream都没有被执行输出操作，那么这些DStream就都不会被求值。如果StreamingContext中没有设定输出操作，整个context就都不会启动。

| Output Operation                            | Meaning                                                      |
| ------------------------------------------- | ------------------------------------------------------------ |
| print()                                     | Prints the  first ten elements of every batch of data in a DStream on the driver node  running the streaming application. This is useful for development and  debugging. **Python API** This is  called **pprint()** in the Python API. |
| **saveAsTextFiles**(*prefix*, [*suffix*])   | Save this  DStream’s contents as text files. The file name at each batch interval is  generated based on *prefix* and *suffix*: *“prefix-TIME_IN_MS[.suffix]”*. |
| **saveAsObjectFiles**(*prefix*, [*suffix*]) | Save this  DStream’s contents as SequenceFiles of serialized Java objects. The file name at each batch interval  is generated based on *prefix* and *suffix*: *“prefix-TIME_IN_MS[.suffix]”*. **Python API** This is not available in the Python API. |
| **saveAsHadoopFiles**(*prefix*, [*suffix*]) | Save this  DStream’s contents as Hadoop files. The file name at each batch interval is  generated based on *prefix* and *suffix*: *“prefix-TIME_IN_MS[.suffix]”*. **Python API** This is not available in the Python API. |
| foreachRDD(*func*)                          | The most  generic output operator that applies a function, *func*, to each RDD generated from the stream. This function should  push the data in each RDD to an external system, such as saving the RDD to  files, or writing it over the network to a database. Note that the function *func* is executed in the driver process  running the streaming application, and will usually have RDD actions in it  that will force the computation of the streaming RDDs. |

## 进阶

### 累加器和广播变量

和RDD中的累加器和广播变量的用法完全一样. 

### DataFrame ans SQL Operations

通过创建一个实例化的SQLContext单实例来实现这个工作

```scala
val spark = SparkSession.builder.config(conf).getOrCreate()
import spark.implicits._
count.foreachRDD(rdd =>{
    val df: DataFrame = rdd.toDF("word", "count")
    df.createOrReplaceTempView("words")
    spark.sql("select * from words").show
})

```

### Caching / Persistence

Streams 同样允许开发者将流数据保存在内存中。也就是说，在DStream 上使用 persist()方法将会自动把DStreams中的每个RDD保存在内存中。

当DStream中的数据要被多次计算时，这个非常有用（如在同样数据上的多次操作）。对于像reduceByWindow和reduceByKeyAndWindow以及基于状态的(updateStateByKey)这种操作，保存是隐含默认的。

