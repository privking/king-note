# RDD

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象。

在代码中是一个抽象类，它代表一个**弹性的、不可变、可分区、里面的元素可并行**计算的集合。

## RDD的5个主要属性

- **A list of partitions**

​       多个分区. 分区可以看成是数据集的基本组成单位.

​       对于 RDD 来说, 每个分区都会被一个计算任务处理, 并决定了并行计算的粒度.

​       用户可以在创建 RDD 时指定 RDD 的分区数, 如果没有指定, 那么就会采用默认值. 默认值就是程序所分配到的 CPU Core 的数目.

​       每个分配的存储是由BlockManager 实现的. 每个分区都会被逻辑映射成 BlockManager 的一个 Block, 而这个 Block 会被一个 Task 负责计算.

- **A function for computing each split**

​       计算每个切片(分区)的函数.

​       Spark 中 RDD 的计算是以分片为单位的, 每个 RDD 都会实现 compute 函数以达到这个目的.

- **A list of dependencies on other RDDs**

​       与其他 RDD 之间的依赖关系

​       RDD 的每次转换都会生成一个新的 RDD, 所以 RDD 之间会形成类似于流水线一样的前后依赖关系. 在部分分区数据丢失时, Spark 可以通过这个依赖关系重新计算丢失的分区数据, 而不是对 RDD 的所有分区进行重新计算.

- **Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)**

​       对存储键值对的 RDD, 还有一个可选的分区器.

​       只有对于 key-value的 RDD, 才会有 Partitioner, 非key-value的 RDD 的 Partitioner 的值是 None. Partitiner 不但决定了 RDD 的本区数量, 也决定了 parent RDD Shuffle 输出时的分区数量.

- **Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)**

​       存储每个切片优先(preferred location)位置的列表. 比如对于一个 HDFS 文件来说, 这个列表保存的就是每个 Partition 所在文件块的位置. 按照“移动数据不如移动计算”的理念, Spark 在进行任务调度的时候, 会尽可能地将计算任务分配到其所要处理数据块的存储位置.

## RDD特点

### 弹性

- 存储的弹性：内存与磁盘的自动切换；
- 容错的弹性：数据丢失可以自动恢复；
- 计算的弹性：计算出错重试机制；
- 分片的弹性：可根据需要重新分片。

### 分区

RDD 逻辑上是分区的，每个分区的数据是抽象存在的，计算的时候会通过一个compute函数得到每个分区的数据。

如果 RDD 是通过已有的文件系统构建，则compute函数是读取指定文件系统中的数据，如果 RDD 是通过其他 RDD 转换而来，则 compute函数是执行转换逻辑将其他 RDD 的数据进行转换。

### 只读

RDD 是只读的，要想改变 RDD 中的数据，只能在现有 RDD 基础上创建新的 RDD。

由一个 RDD 转换到另一个 RDD，可以通过丰富的转换算子实现，不再像 MapReduce 那样只能写map和reduce了。

RDD 的操作算子包括两类，

- 一类叫做transformation，它是用来将 RDD 进行转化，构建 RDD 的血缘关系；
- 另一类叫做action，它是用来触发 RDD 进行计算，得到 RDD 的相关计算结果或者 保存 RDD 数据到文件系统中.

### 依赖(血缘)

RDDs 通过操作算子进行转换，转换得到的新 RDD 包含了从其他 RDDs 衍生所必需的信息，RDDs 之间维护着这种血缘关系，也称之为依赖。

依赖包括两种，

- 一种是窄依赖，RDDs 之间分区是一一对应的，
- 另一种是宽依赖，下游 RDD 的每个分区与上游 RDD(也称之为父RDD)的每个分区都有关，是多对多的关系。

![image-20210710195537806](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210710195537806-1625918144-13245d.png)

### 缓存

如果在应用程序中多次使用同一个 RDD，可以将该 RDD 缓存起来，该 RDD 只有在第一次计算的时候会根据血缘关系得到分区的数据，在后续其他地方用到该 RDD 的时候，会直接从缓存处取而不用再根据血缘关系计算，这样就加速后期的重用。

### checkpoint

虽然 RDD 的血缘关系天然地可以实现容错，当 RDD 的某个分区数据计算失败或丢失，可以通过血缘关系重建。

但是对于长时间迭代型应用来说，随着迭代的进行，RDDs 之间的血缘关系会越来越长，一旦在后续迭代过程中出错，则需要通过非常长的血缘关系去重建，势必影响性能。

为此，RDD 支持checkpoint 将数据保存到持久化的存储中，这样就可以切断之前的血缘关系，因为checkpoint 后的 RDD 不需要知道它的父 RDDs 了，它可以从 checkpoint 处拿到数据。