# SparkSql

Spark SQL 是 Spark 用于结构化数据(structured data)处理的 Spark 模块.

Spark SQL 它提供了2个编程抽象, 类似 Spark Core 中的 RDD

- DataFrame
- DataSet

**Integrated(易整合)**

无缝的整合了 SQL 查询和 Spark 编程

![image-20210717211055900](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717211055900-1626527463-088658.png)

**Uniform Data Access(统一的数据访问方式)**

使用相同的方式连接不同的数据源.

![image-20210717211117615](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717211117615-1626527477-116fdd.png)

**Hive Integration(集成 Hive)**

在已有的仓库上直接运行 SQL 或者 HiveQL

![image-20210717211139911](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717211139911-1626527500-20d7b7.png)

**Standard Connectivity(标准的连接方式)**

通过 JDBC 或者 ODBC 来连接

![image-20210717211208319](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717211208319-1626527528-758788.png)

## DataFrame

与 RDD 类似，DataFrame 也是一个分布式数据容器。

然而DataFrame更像传统数据库的二维表格，除了数据以外，还记录数据的结构信息，即schema。

同时，与Hive类似，DataFrame也支持嵌套数据类型（struct、array和map）。

![image-20210717211328377](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717211328377-1626527608-a3009e.png)

左侧的RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解Person类的内部结构。

而右侧的DataFrame却提供了详细的结构信息，使得 Spark SQL 可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。

DataFrame是为数据提供了Schema的视图。可以把它当做数据库中的一张表来对待，

DataFrame也是懒执行的

性能上比 RDD要高，主要原因： 优化的执行计划：查询计划通过Spark catalyst optimiser进行优化。

## DataSet

- 是DataFrame API的一个扩展，是 SparkSQL 最新的数据抽象(1.6新增)。
- 用户友好的API风格，既具有类型安全检查也具有DataFrame的查询优化特性。
- Dataset支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。
-  样例类被用来在DataSet中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。
- DataFrame是DataSet的特列，DataFrame=DataSet[Row] ，所以可以通过as方法将DataFrame转换为DataSet。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息都用Row来表示。
- DataSet是强类型的。比如可以有DataSet[Car]，DataSet[Person].
- DataFrame只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，比如你可以对一个String进行减法操作，在执行的时候才报错，而DataSet不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查。就跟JSON对象和类对象之间的类比。

## SparkSql编程

### SparkSession

在老的版本中，SparkSQL 提供两种 SQL 查询起始点：一个叫SQLContext，用于Spark 自己提供的 SQL 查询；一个叫 HiveContext，用于连接 Hive 的查询。

从2.0开始, SparkSession是 Spark 最新的 SQL 查询起始点，实质上是SQLContext和HiveContext的组合，所以在SQLContext和HiveContext上可用的 API 在SparkSession上同样是可以使用的。

**SparkSession内部封装了SparkContext，所以计算实际上是由SparkContext完成的**。

当我们使用 spark-shell 的时候, spark 会自动的创建一个叫做spark的SparkSession, 就像我们以前可以自动获取到一个sc来表示SparkContext

### DataFrame编程

#### 创建DataFrame

- 从RDD转换得到
- 通过数据源得到
  - jdbc
  - hive
  - parquet
  - json
  - scala集合

![image-20210717221425215](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717221425215-1626531265-769d95.png)

#### 创建临时表

![image-20210717221710531](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717221710531-1626531430-f51c1f.png)

**createGlobalTempView**

创建全局临时表

![image-20210717222225021](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717222225021-1626531745-4e3604.png)

**createOrReplaceTempView**

创建或替换已有临时表

![image-20210717222041193](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717222041193-1626531641-5e5cff.png)

**createTempView**

创建临时表

#### DSL风格

![image-20210717222738605](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210717222738605-1626532058-6fe91f.png)

$相当于是查询列名（column）的函数

也可以写成`fileter("age>20"),select($"name",$"age")`

#### RDD转DF

```scala
object RDDToDF {

  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession.builder()
      .appName("CreateDF")
      .master("local[2]")
      .getOrCreate()

    val sc =spark.sparkContext

    val rdd = sc.parallelize(List(User("kk", 11),
      User("lambda", 12),
        User("king", 13)
    ))
    //这个spark为上面创建出来的sparkSession
     //必须隐式转换
    import spark.implicits._

    //val df = rdd.toDF("name","age")

    //User为样例类，所以可以直接toDF
    val df = rdd.toDF
      
    //直接从集合得到DF,也必须要隐式转换
    //val df = (1 to 10).toDF



    df.createOrReplaceTempView("people")

    spark.sql("select * from people where age > 11").show

    spark.stop()
  }
}

case class User(name:String,age:Int)
```

```scala
//方式二

def run2(sparkSession: SparkSession)={
		//隐式转换
      import sparkSession.implicits._
      
      import org.apache.spark.sql.types._
      //接收数据
      val rdd: RDD[String] = sparkSession.sparkContext.textFile("data/people.txt")
      //第一步：将string类型的rdd转换成row类型的rdd
      val row: RDD[Row] = rdd.map(_.split(","))
      	.map(x => Row(x(0), x(1).trim.toInt))

		//第二部:使用StructType 创建一个schema信息
      val struct: StructType = StructType(
      	//StructType里放StructField,有三个参数
      	//1.字段名
      	//2.type类型的字段类型
      	//3.是否为空
        StructField("name", StringType, false) ::
          StructField("age", IntegerType, true) :: Nil
      )
      //第三部结合一二两步
      val df: DataFrame = sparkSession.createDataFrame(row, struct)
      df.show()
    }
```

#### DF转RDD

```scala
df.rdd
```

### DataSet编程

DataSet 和 RDD 类似, 但是DataSet没有使用 Java 序列化或者 Kryo序列化, 而是使用一种专门的编码器去序列化对象, 然后在网络上处理或者传输.

DataSet是具有强类型的数据集合，需要提供对应的类型信息。

#### 创建DS

通过scala集合得到

```scala
def main(args: Array[String]): Unit = {
  val spark: SparkSession = SparkSession.builder()
    .appName("CreateDS")
    .master("local[2]")
    .getOrCreate()
  import spark.implicits._
  val ds = List(
    User("kk", 11),
    User("lambda", 12),
    User("king", 13))
    .toDS
  //ds.show
  ds.createOrReplaceTempView("user")
  spark.sql("select * from user where age>11").show


  spark.close
}
```

通过rdd得到

通过df得到

#### DS转RDD

```scala
ds.rdd
//得到的不是RDD[ROW],而是强类型
```

#### DF转DS

```scala
//User为样例类
//需要隐式转换
import spark.implicits._
val ds = df.as[User]
```

#### DS转DF

```scala
//不需要隐式转换
val df = ds.toDF
```

### RDD, DataFrame和 DataSet 之间的关系

在 SparkSQL 中 Spark 为我们提供了两个新的抽象，分别是DataFrame和DataSet。他们和RDD有什么区别呢？首先从版本的产生上来看：

RDD (Spark1.0) —> Dataframe(Spark1.3) —> Dataset(Spark1.6)

如果同样的数据都给到这三个数据结构，他们分别计算之后，都会给出相同的结果。不同是的他们的执行效率和执行方式。

![image-20210718121207860](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210718121207860-1626581535-d130ae.png)

#### 共性

1. RDD、DataFrame、Dataset全都是 Spark 平台下的分布式弹性数据集，为处理超大型数据提供便利
2. 三者都有惰性机制，在进行创建、转换，如map方法时，不会立即执行，只有在遇到Action如foreach时，三者才会开始遍历运算。
3. 三者都会根据 Spark 的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出
4. 三者都有partition的概念
5. 三者有许多共同的函数，如map, filter，排序等
6. 在对 DataFrame和Dataset进行操作许多操作都需要这个包进行支持 import spark.implicits._
7. DataFrame和Dataset均可使用模式匹配获取各个字段的值和类型

### 自定义UDAF

```scala
object UDAFDemo {
  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession.builder()
      .appName("UDAFDemo")
      .master("local[2]")
      .getOrCreate()

    val df = spark.read.json("D:\\IDEA\\spark-study\\spark-sql\\src\\main\\resources\\people.json")
    df.createOrReplaceTempView("people")
    spark.udf.register("mySum",new MySum)
    spark.sql("select mySum(age) from people").show
  }
}

class MySum extends UserDefinedAggregateFunction{
  //定义输入数据类型
  override def inputSchema: StructType =
    StructType(StructField("ele",LongType)::Nil)

  //缓冲区的数据类型
  override def bufferSchema: StructType =
    StructType(StructField("sum",LongType)::Nil)

  //聚合结果数据类型
  override def dataType: DataType = LongType

  //相同的输入是否返回相同输出
  override def deterministic: Boolean = true

  //对缓冲区初始化
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    //初始化和
    //相当于0下标位置初始化为0
    buffer(0) = 0L //buffer.update(0,0L)
  }

  //分区内聚合
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {

    if (!input.isNullAt(0)) {
      //input相当于使用聚合函数时传的参数
      val v = input.getLong(0)
      buffer(0) = buffer.getLong(0)+v
    }

  }

  //分区间聚合
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit =
    buffer1(0) = buffer1.getLong(0) +buffer2.getLong(0)


  //最终输出
  override def evaluate(buffer: Row): Any = buffer.getLong(0)

}
```

### 自定义UDF

```scala
spark.udf.register("toUpper", (s: String) => s.toUpperCase)
```

