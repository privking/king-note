# RDD函数中的传递

 Spark 进行编程的时候, 初始化工作是在 driver端完成的, 而实际的运行程序是在executor端进行的. 所以就涉及到了进程间的通讯, 数据是需要序列化的.

```scala
object SerDemo {
    def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setAppName("SerDemo").setMaster("local[*]")
        val sc = new SparkContext(conf)
        val rdd: RDD[String] = sc.parallelize(Array("hello world", "hello atguigu", "atguigu", "hahah"), 2)
        val searcher = new Searcher("hello")
        val result: RDD[String] = searcher.getMatchedRDD1(rdd)
        result.collect.foreach(println)
    }
}
//需求: 在 RDD 中查找出来包含 query 子字符串的元素

// query 为需要查找的子字符串
class Searcher(val query: String){
    // 判断 s 中是否包括子字符串 query
    def isMatch(s : String) ={
        s.contains(query)
    }
    // 过滤出包含 query字符串的字符串组成的新的 RDD
    //需要传递isMatch
    def getMatchedRDD1(rdd: RDD[String]) ={
        rdd.filter(isMatch)  //
    }
    // 过滤出包含 query字符串的字符串组成的新的 RDD
    //query依然是类属性，所以需要传递类反序列化后解析
    def getMatchedRDD2(rdd: RDD[String]) ={
        rdd.filter(_.contains(query))
    }
    
    // 过滤出包含 query字符串的字符串组成的新的 RDD
    //传递局部变量q,只需要q实现了序列化
    def getMatchedRDD3(rdd: RDD[String]) ={
        val q = query
        rdd.filter(_.contains(q))
    }
}
//常用的函数以及基本类型都实现了序列化
//getMatchedRDD1,getMatchedRDD2需要Searcher实现Serializable
//getMatchedRDD3需要局部变量实现序列化，而String这种已经实现了序列化
```

## kryo序列化框架

Java 的序列化比较重, 能够序列化任何的类. 比较灵活,但是相当的慢, 并且序列化后对象的体积也比较大.

Spark 出于性能的考虑, 支持另外一种序列化机制: kryo (2.0开始支持). kryo 比较快和简洁.(速度是Serializable的10倍). 想获取更好的性能应该使用 kryo 来序列化.

从2.0开始, Spark 内部已经在使用 kryo 序列化机制: 当 RDD 在 Shuffle数据的时候, 简单数据类型, 简单数据类型的数组和字符串类型已经在使用 kryo 来序列化.

有一点需要注意的是: 即使使用 kryo 序列化, **也要继承 Serializable 接口.**

```scala
 val conf: SparkConf = new SparkConf()
            .setAppName("SerDemo")
            .setMaster("local[*]")
            // 替换默认的序列化机制 可以省(如果调用registerKryoClasses
            .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
            // 注册需要使用 kryo 序列化的自定义类
            .registerKryoClasses(Array(classOf[Searcher]))

```

