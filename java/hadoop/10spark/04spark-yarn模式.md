# Spark-yarn模式

## 配置

### yarn-site.xml

```xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>

```

### spark-default.conf

```properties
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://node1:9000/spark-job-log-yarn
#直接跳转到spark history
spark.yarn.historyServer.address=node1:18080
spark.history.ui.port=18080
```

### spark-env.conf

```properties
YARN_CONF_DIR=/usr/local/soft/hadoop2.7/hadoop-2.7.2/etc/hadoop

export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=30 -Dspark.history.fs.logDirectory=hdfs://node1:9000/spark-job-log-yarn"

```

## 几种模式对比

| 模式       | Spark安装机器数 | 需启动的进程   | 所属者 |
| ---------- | --------------- | -------------- | ------ |
| Local      | 1               | 无             | Spark  |
| Standalone | 多台            | Master及Worker | Spark  |
| Yarn       | 1               | Yarn及HDFS     | Hadoop |
| Mesos      |                 |                |        |

