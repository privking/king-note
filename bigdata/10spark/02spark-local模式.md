# SparkLocal模式

## 计算PI

```bash
>bin/spark-submit \
 --class org.apache.spark.examples.SparkPi \
 --master local[2] \
 ./examples/jars/spark-examples_2.11-2.1.1.jar 100
```

- local[2] 模拟两个线程
- 100 迭代次数，参数

## 语法

```bash
./bin/spark-submit \
 --class <main-class> \
 --master <master-url> \
 --deploy-mode <deploy-mode> \
 --conf <key>=<value> \
 ... # other options
 <application-jar> \
 [application-arguments]
```

-  --master 指定 master 的地址，默认为local. 表示在本机运行.
- --class 你的应用的启动类 (如 org.apache.spark.examples.SparkPi)
- --deploy-mode 是否发布你的驱动到 worker节点(cluster 模式) 或者作为一个本地客户端 (client 模式) (default: client)
- --conf: 任意的 Spark 配置属性， 格式key=value. 如果值包含空格，可以加引号"key=value"
- --deploy-mode：driver的位置，默认client。Distinguishes where the driver process runs. In “cluster” mode, the framework launches the driver inside of the cluster. In “client” mode, the submitter launches the driver outside of the cluster.
- application-jar: 打包好的应用 jar,包含依赖. 这个 URL 在集群中全局可见。 比如**hdfs:// 共享存储系统**， 如果是 file:// path， 那么所有的节点的path**都包含同样的jar**
- application-arguments: 传给main()方法的参数
- --executor-memory 1G 指定每个executor可用内存为1G
- --total-executor-cores 6 指定所有executor使用的cpu核数为6个
- --executor-cores 表示每个executor使用的 cpu 的核数



### More Actions  Master URL  

| Master URL        | Meaning                                                      |
| ----------------- | ------------------------------------------------------------ |
| local             | Run Spark  locally with one worker thread (i.e. no parallelism at all). |
| local[K]          | Run Spark  locally with K worker threads (ideally, set this to the number of cores on  your machine). |
| local[*]          | Run Spark  locally with as many worker threads as logical cores on your machine. |
| spark://HOST:PORT | Connect to the  given Spark   standalone clustermaster. The port must be whichever one your master is  configured to use, which is 7077 by default. |
| mesos://HOST:PORT | Connect to the  given Mesos cluster. The port must be whichever one your is configured to use, which is  5050 by default. Or, for a Mesos cluster using ZooKeeper, use mesos://zk://.... To submit with --deploy-mode cluster, the HOST:PORT  should be configured to connect to the MesosClusterDispatcher |
| yarn              | Connect to a YARN cluster  in client or cluster mode depending on the value of  --deploy-mode. The  cluster location will be found based on the HADOOP_CONF_DIR or YARN_CONF_DIR variable. |





## Spark-shell

### 本地模式使用shell

```sh
bin/spark-shell  --master local[2]
```

### 计算world-count

```spark
sc.textFile("king-test/").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _).collect
```

![image-20210709222446252](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210709222446252-1625840693-373f60.png)

在的wordcount案例集中, spark-shell 就是我们的驱动程序, 所以我们可以在其中键入我们任何想要的操作, 然后由他负责发布.

驱动程序通过SparkContext对象来访问 Spark, SparkContext对象相当于一个到 Spark 集群的连接.

在 spark-shell 中, 会自动创建一个SparkContext对象, 并把这个对象命名为sc.

