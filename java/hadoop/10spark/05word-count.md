# word-count

## 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.11</artifactId>
        <version>2.1.1</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <!-- 打包插件, 否则 scala 类不会编译并打包进去 -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.4.6</version>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## scala

```scala
object Hello {
  def main(args: Array[String]): Unit = {
    //创建SparkContext
    //本地模式需要设置Master
    //打包集群运行的时候需要去掉setMaster,在运行时 --master
    val conf = new SparkConf().setMaster("local[2]").setAppName("Hello")
    val sc = new SparkContext(conf)

    //得到RDD
    val lineRDD = sc.textFile(args(0))
    //RDD数据操作
    val resultRDD = lineRDD.flatMap(_.split(" "))
      .map((_, 1))
      .reduceByKey(_ + _)
    //执行行动算子
    val worldCountArr = resultRDD.collect()
    //关闭SparkContext
    worldCountArr.foreach(println)
    sc.stop()
  }
}
```

