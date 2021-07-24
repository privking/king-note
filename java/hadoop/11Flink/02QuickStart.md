# QuackStart

## 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-scala_2.11</artifactId>
        <version>1.10.0</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-streaming-scala -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-scala_2.11</artifactId>
        <version>1.10.0</version>
    </dependency>
</dependencies>


<build>
    <plugins>
        <!-- 该插件用于将Scala代码编译成class文件 -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.4.6</version>
            <executions>
                <execution>
                    <!-- 声明绑定到maven的compile阶段 -->
                    <goals>
                        <goal>compile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 批处理方式

```scala
def main(args: Array[String]): Unit = {
  //创建执行环境
  val env = ExecutionEnvironment.getExecutionEnvironment
  //读取数据
  val dataSet: DataSet[String] = env.readTextFile("D:\\IDEA\\flink-study\\first\\src\\main\\resources\\1.txt")
  val res: DataSet[(String, Int)] = dataSet
    //需要导入隐式 org.apache.flink.api.scala._
    //底层将所有类型包装成 TypeInformation
    .flatMap(_.split(" "))
    .map((_, 1))
    .groupBy(0)  //按照元组第一个进行分组,可以传多个参数
    .sum(1)   //按照元组第二个进行求和聚合
  res.print()
}
```

## 流处理方式

```scala
object StreamWordCount {
  def main(args: Array[String]): Unit = {
    //创建流式执行环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(8) //可以设置并行度

    //flink提供的工具类
    //可以从args,也可以从map,也可以从properties
    val param: ParameterTool = ParameterTool.fromArgs(args)
    val host = param.get("host")
    val port = param.getInt("port")

    val stream = env.socketTextStream(host, port)
    stream.flatMap(_.split(" "))
      .map((_, 1))
      .keyBy(0)
      .sum(1)
      .print()
      .setParallelism(1) //也可以设置并行度

    env.execute("stream word count job")
  }
}
```

