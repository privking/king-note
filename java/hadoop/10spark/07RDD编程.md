# RDD编程

## 创建RDD

### parallelize

```scala
//1.sparkContext
val conf = new SparkConf().setMaster("local[*]").setAppName("CreateRDD")
val sc = new SparkContext(conf)
//2.创建RDD
//从scala集合得到rdd
val arr = Array(1,2,3,5,8,6,7)
//parallelize有默认参数，切片数量sc.parallelize(arr,2)
//sc.makeRDD(arr,2) makeRdd与parallelize区别不大
val rdd = sc.parallelize(arr)
```

### makeRdd

```scala
//1.sparkContext
val conf = new SparkConf().setMaster("local[*]").setAppName("CreateRDD")
val sc = new SparkContext(conf)
//2.创建RDD
//从scala集合得到rdd
val arr = Array(1,2,3,5,8,6,7)
//sc.makeRDD(arr,2) makeRdd与parallelize区别不大
val rdd = sc.makeRDD(arr,2) 
```

- 一旦 RDD 创建成功, 就可以通过并行的方式去操作这个分布式的数据集了.
- parallelize和makeRDD还有一个重要的参数就是把数据集**切分成的分区数**.
- Spark 会为每个分区运行一个任务(task). 正常情况下, Spark 会自动的根据你的集群来设置分区数

### 外部创建RDD

Spark 也可以从任意 Hadoop 支持的存储数据源来创建分布式数据集.

可以是本地文件系统, HDFS, Cassandra, HBase, Amazon S3 等等.

Spark 支持 文本文件, SequenceFiles, 和其他所有的 Hadoop InputFormat.

```scala
var distFile = sc.textFile("words.txt")
```

- url可以是本地文件系统文件, hdfs://..., s3n://...等等
- 如果是使用的本地文件系统的路径, 则必须**每个节点**都要存在这个路径
- 所有基于文件的方法, 都支持目录, 压缩文件, 和通配符(`*`). 例如: textFile("/my/directory"), textFile("/my/directory/*.txt"), and textFile("/my/directory/`*`.gz").
- textFile还可以有第二个参数, 表示**分区数**. 默认情况下, 每个块对应一个分区.(对 HDFS 来说, 块大小默认是 128M). 可以传递一个**大于块数的分区数, 但是不能传递一个比块数小的分区数**.

## RDD转换

在 RDD 上支持 2 种操作:transformation和action

在 Spark 中几乎所有的transformation操作都是懒执行的(lazy), 也就是说transformation操作并不会立即计算他们的结果, 而是记住了这个操作.

只有当通过一个action来获取结果返回给驱动程序的时候这些转换操作才开始计算.

但是我们可以通过`persist (or cache)`方法来持久化一个 RDD 在内存中, 也可以持久化到磁盘上, 来加快访问速度. 

### map

对每个元素都执行map操作

```scala
val rdd2 = rdd1.map(_ * 2)
```

### mapPartitions

mapPartitions 传入参数是一个迭代器，包含一个分区的数据，有几个分区mapPartitions就执行几次
返回也是一个迭代器

```scala
val rdd2 = rdd.mapPartitions(it=>it.map(_*2))
//it是传入的参数
//it.map是对每个分区里面的据进行map,是scala的map
//每次处理一个分区的数据，这个分区的数据处理完后，原 RDD 中该分区的数据才能释放，可能导致 OOM
```

### mapPartitionsWithIndex

mapPartitionsWithIndex可以得到分区序号

```scala
val rdd2 = rdd.mapPartitionsWithIndex((index, it) => it.map((index, _)))
```

### flatMap

将集合展开，将集合拍平

```scala
//2.创建RDD
val list1 =List(1 ,2,3 ,4 ,5 )
val rdd= sc.parallelize(list1)
//3.转换
//传出参数必须是集合,rdd2中是返回的所有list元素聚合到一起
val rdd2 = rdd.flatMap(x=>List(x,x+1,x+2))
//4.行动算子
val ints = rdd2.collect()
```

### filter

```scala
val rdd2 = rdd.filter(_>2)
```

### glom

每一个分区的元素合并成一个数组，形成新的 RDD 类型是RDD[Array[T]]

结果例如：Array(Array(10), Array(20, 30), Array(40), Array(50, 60))

```scala
val rdd2 = rdd.glom()
```

### groupBy

分组

This operation may be very expensive. If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using PairRDDFunctions.aggregateByKey or PairRDDFunctions.reduceByKey will provide much better performance

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val rdd= sc.parallelize(list1,3)

val rdd2: RDD[(Int, Iterable[Int])] = rdd.groupBy(_%2)
```

### sample

随机抽样  `sample(withReplacement, fraction, seed)`

- 以指定的随机种子随机抽样出比例为fraction的数据，(抽取到的数量是: size * fraction). 需要注意的是得到的结果并不能保证准确的比
- withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样. **放回表示数据有可能会被重复抽取到**, false 则不可能重复抽取到. 如果是**false, 则fraction必须是:[0,1], 是 true 则大于等于0,大于1表示几倍**就可以了.
- seed用于指定随机数生成器种子。 一般用默认的, 或者传入当前的时间戳

### distinct

对 RDD 中元素执行去重操作. 参数表示任务的数量.默认值和分区数保持一致

### coalesce

缩减分区数到指定的数量，用于大数据集过滤后，提高小数据集的执行效率

默认不进行shuffle,第二个参数可以控制是否shuffle,第三个参数为分区器（默认hash）

默认不能增加分区，加上shuffle可以增加

```scala
rdd1.coalesce(2)
```

### repartition

根据新的分区数, 重新 shuffle 所有的数据, 这个操作总会通过网络.

新的分区数相比以前可以多, 也可以少

coalesce重新分区，可以选择是否进行shuffle过程。由参数shuffle: Boolean = false/true决定。

repartition实际上是调用的coalesce，进行shuffle

### sortBy

排序，默认正序

sortBy(func,[ascending], [numTasks])  ascending是否升序

```scala
rdd1.sortBy(x => x).collect
```

