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

## RDD转换（单value）

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

### pipe

pipe(command, [envVars])

管道，针对每个分区，把 RDD 中的每个数据通过管道传递给shell命令或脚本，返回输出的RDD。一个分区执行一次这个命令. 如果只有一个分区, 则执行一次命令.

脚本要放在 worker 节点可以访问到的位置

## RDD转换（双value）

### union

求并集. 对源 RDD 和参数 RDD 求并集后返回一个新的 RDD

**不去重**

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val list2 =List(3 ,4 ,5 ,6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)
val rdd3 = rdd.union(rdd2)
//rdd3=rdd++rdd2
```

可以使用`++`实现相同功能

### subtract 

计算差集. 从原 RDD 中减去 原 RDD 和 otherDataset 中的共同的部分，**otherDataset 有原RDD没有的也不计入结果**

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val list2 =List(3 ,4 ,5 ,6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)
val rdd3 = rdd.subtract(rdd2)
//结果为1,2
```

### intersection

计算交集. 对源 RDD 和参数 RDD 求交集后返回一个新的 RDD

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val list2 =List(3 ,4 ,5 ,6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)
val rdd3 = rdd.intersection(rdd2)
//3,4,5
```

### cartesian

计算 2 个 RDD 的笛卡尔积. 尽量避免使用

```scala
val list1 =List(1 ,2 )
val list2 =List(6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)
val rdd3 = rdd.cartesian(rdd2)
//(1,6)
//(1,7)
//(2,6)
//(2,7)
```

### zip

拉链操作. 需要注意的是, 在 Spark 中, 两个 RDD 的**元素的数量**和**分区数**都必须相同, 否则会抛出异常.(在 scala 中, 两个集合的长度可以不同)

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val list2 =List(3 ,4 ,5 ,6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)
val rdd3 = rdd.zip(rdd2)
//(1,3)
//(2,4)
//(3,5)
//(4,6)
//(5,7)
```

### zipPartitions

分区与分区之间拉，只要求分区数相同即可。

```scala
val list1 =List(1 ,2,3 ,4 ,5 )
val list2 =List(3 ,4 ,5 ,6,7)
val rdd= sc.parallelize(list1)
val rdd2= sc.parallelize(list2)

//分区与分区之间拉
val rdd3 = rdd.zipPartitions(rdd2)((it1,it2)=>{
  it1.zip(it2)// 使用scala的zip，按照短的拉，多的丢弃
  //it1.zipAll(it2,100,200) //按照长的集合拉，不够的使用默认值
})
```

### zipWithIndex

与下标zip,返回元组，第二个为下标，第一个为值

```scala
val rdd3: RDD[(Int, Long)] = rdd.zipWithIndex()
```

## RDD转换(Key-Value)

大多数的 Spark 操作可以用在任意类型的 RDD 上, 但是有一些比较特殊的操作只能用在key-value类型的 RDD 上.

这些特殊操作大多都涉及到 shuffle 操作, 比如: 按照 key 分组(group), 聚集(aggregate)等.

### partitionBy

对 pairRDD 进行分区操作，如果原有的 partionRDD 的分区器和传入的分区器**相同, 则返回原 pairRDD**，否则会**生成 ShuffleRDD**，即会产生 shuffle 过程

产生shuffle就相当于一个新的阶段（stage）,一个阶段内都是并行执行，互相不影响

```scala
val list1 =List(1 ,2,3 ,4 ,5,5,6 )
val rdd= sc.parallelize(list1,3)
val rdd2 = rdd.map((_,1))
val rdd3 = rdd2.partitionBy(new HashPartitioner(2))
val ints = rdd3.collect()
```

### reduceByKey

将相同key的value聚合到一起

在shuffle之前有combine（预聚合）操作（**所有的聚合算子都有预聚合,调用底层函数的时候有默认参数mapSideConbine=true**）

```scala
val rdd2 = rdd.map((_,1))
val rdd3 = rdd2.reduceByKey(_+_)
val ints = rdd3.collect()
//word count
```

### groupByKey

按照key进行分组.

只能用于keyValue类型，groupBy可以用于任意类型

```scala
val rdd2 = rdd.map((_,1))
val rdd3 = rdd2.groupByKey()
val ints = rdd3.collect()
```

### foldByKey

按key进行折叠

```scala
val list1 =List(1 ,2,3 ,4 ,5,5,6 )
val rdd= sc.parallelize(list1,3)
val rdd2 = rdd.map((_,1))
val rdd3 = rdd2.foldByKey(0)(_+_)
val ints = rdd3.collect()
```

### aggregateByKey

相当于foldByKey和groupBy的结合

在shuffle前（分区内）进行foldByKey

在shuffle后（分区间）进行groupBy

能够实现分区内和分区间的执行逻辑不同（foldByKey和groupByKey都是相同的）

```scala
rdd.aggregateByKey(Int.MinValue)(math.max(_, _), _ +_)
//取出每个区间相同key的最大值，然后分区间最大值相加
```

```scala
//求每个key出现的次数以及key相同的value的和
val list1 =List(("a",1) ,("b",2),("c",3),("a",1),("b",2),("c",3),("d",4) )
val rdd= sc.parallelize(list1,3)
val rdd2 = rdd.aggregateByKey((0,0))(
  {
    case ((sum,count),v)=>(sum+v,count+1)
  },{
    case ((sum1,count1),(sum2,count2))=>(sum1+sum2,count1+count2)
  }
)
val ints = rdd2.collect()


//每个key的平均值
 val list1 =List(("a",1) ,("b",2),("c",3),("a",1),("b",2),("c",3),("d",4) )
    val rdd= sc.parallelize(list1,3)
    val rdd2 = rdd.aggregateByKey((0,0))(
      {
        case ((sum,count),v)=>(sum+v,count+1)
      },{
        case ((sum1,count1),(sum2,count2))=>(sum1+sum2,count1+count2)
      }
    ).map{
      case (key,(sum,count)) => (key,sum.toDouble/count)
    }
```

### combineByKey

在aggregateByKey基础上，初始值根据第一个key的value生成

```scala
val rdd2 = rdd.combineByKey(
  v=>v,     //初始值计算            
  (c:Int,v)=>c+v, //分区前执行的函数，需要类型显式指定
  (c1:Int,c2:Int)=>c1+c2 //分区间执行的函数
)
```

### 4种聚合函数的区别

- reduceByKey
  - 分区内聚合和分区间聚合逻辑相同
- foldByKey
  - 分区内和分区间聚合逻辑相同，但是有初始值
- aggregateByKey
  - 有初始值，分区内和分区间逻辑可以不同
- combineByKey
  - 可以根据key的第一个value计算初始值，分区内和分区间逻辑可以不同

4个函数底层都调用了combineByKeyWithClassTag

### sortByKey

根据key排序，可以指定升序降序，默认升序

```scala
val list1 =List(("a",1) ,("b",2),("c",3),("a",1),("b",2),("c",3),("d",4) )
val rdd= sc.parallelize(list1,3)
val rdd2 = rdd.sortByKey(true)
```

### mapValues

.....

### cogroup

```scala
 val rdd1 = sc.parallelize(Array((1, 10),(2, 20),(1, 100),(3, 30)),1)
val rdd2 = sc.parallelize(Array((1, "a"),(2, "b"),(1, "aa"),(3, "c")),1)
rdd1.cogroup(rdd2).collect
//Array((1,(CompactBuffer(10, 100),CompactBuffer(a, aa))), (3,(CompactBuffer(30),CompactBuffer(c))), (2,(CompactBuffer(20),CompactBuffer(b))))
```

### join

内连接:

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素对在一起的(K,(V,W))的RDD

```scala
 var rdd1 = sc.parallelize(Array((1, "a"), (1, "b"), (2, "c")))
  var rdd2 = sc.parallelize(Array((1, "aa"), (3, "bb"), (2, "cc")))
   rdd1.join(rdd2).collect
   // Array((2,(c,cc)), (1,(a,aa)), (1,(b,aa)))
```

如果某一个 RDD 有重复的 Key, 则会分别与另外一个 RDD 的相同的 Key进行组合

也支持外连接: leftOuterJoin, rightOuterJoin, and fullOuterJoin.

### repartitionAndSortWithinPartitions

分区，并且分区后key有序

