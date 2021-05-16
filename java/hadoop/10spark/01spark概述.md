# Spark概述

## 什么是spark

spark是一个快速（基于内存），通用，可扩展的集群计算引擎

## spark特点

### 快速

与 Hadoop 的 MapReduce 相比, Spark 基于内存的运算是 MapReduce 的 100 倍.基于硬盘的运算也要快 10 倍以上.

Spark 实现了高效的 DAG 执行引擎, 可以通过基于内存来高效处理数据流

### 易用

Spark 支持 Scala, Java, Python, R 和 SQL 脚本, 并提供了超过 80 种高性能的算法, 非常容易创建并行 App

而且 Spark 支持交互式的 Python 和 Scala 的 shell, 这意味着可以非常方便地在这些 shell 中使用 Spark 集群来验证解决问题的方法, 而不是像以前一样 需要打包, 上传集群, 验证等. 这对于原型开发非常重要.

### 通用

Spark 结合了SQL, Streaming和复杂分析.

Spark 提供了大量的类库, 包括 SQL 和 DataFrames, 机器学习(MLlib), 图计算(GraphicX), 实时流处理(Spark Streaming) .

可以把这些类库无缝的柔和在一个 App 中.

减少了开发和维护的人力成本以及部署平台的物力成本

#### 可融合性

Spark 可以非常方便的与其他开源产品进行融合.

比如, Spark 可以使用 Hadoop 的 YARN 和 Appache Mesos 作为它的资源管理和调度器, 并且可以处理所有 Hadoop 支持的数据, 包括 HDFS, HBase等

## Spark 内置模块介绍

![image-20210516180955084](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210516180955084-1621159802-727686.png)

#### 集群管理器(Cluster Manager)

Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。

为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器(Cluster Manager)上运行，目前 Spark 支持 4种集群管理器:

1. Hadoop YARN(在国内使用最广泛)

2. Apache Mesos(国内使用较少, 国外使用较多)

3. Standalone(Spark 自带的资源调度器, 需要在集群中的每台节点上配置 Spark)
4. **spark2.3**后支持 k8s

### SparkCore

实现了 Spark 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。SparkCore 中还包含了对弹性分布式数据集(Resilient Distributed DataSet，简称RDD)的API定义

### Spark SQL

是 Spark 用来操作结构化数据的程序包。通过SparkSql，我们可以使用 SQL或者Apache Hive 版本的 SQL 方言(HQL)来查询数据。Spark SQL 支持多种数据源，比如 Hive 表、Parquet 以及 JSON 等

### Spark Streaming

是 Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应。

### Spark MLlib

提供常见的机器学习 (ML) 功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。

