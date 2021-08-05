# Spark Stand Alone

## 配置

### spark-env.sh

```sh
SPARK_MASTER_HOST=hadoop201
SPARK_MASTER_PORT=7077 # 默认端口就是7077, 可以省略不配
```

### slaves

```
node1
node2
node3
```

### 分发到集群

### 启动集群

```sh
sbin/start-all.sh
```

### 计算PI

```
bin/spark-submit \
 --class org.apache.spark.examples.SparkPi \
 --master spark://node1:7077 \
 ./examples/jars/spark-examples_2.11-2.1.1.jar 100
```

## 配置历史服务器

在 Spark-shell 没有退出之前, 我们是可以看到正在执行的任务的日志情况. 但是退出 Spark-shell 之后, 执行的所有任务记录全部丢失.

#### spark-defaults.conf

```
spark.eventLog.enabled      true
spark.eventLog.dir        hdfs://node1:9000/spark-job-log
```

注意:

hdfs://node1:9000/spark-job-log 目录必须**提前存在**, 名字随意

#### spark-env.sh

```
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=30 -Dspark.history.fs.logDirectory=hdfs://node1:9000/spark-job-log"
```

#### 分发配置文件

#### 启动

先启动hdfs

```
sbin/start-history-server.sh
```

