# RDD血缘关系

![image-20210713202537167](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210713202537167-1626179137-2cf8f2.png)

## 窄依赖

如果 B RDD 是由 A RDD 计算得到的, 则 B RDD 就是 Child RDD, A RDD 就是 parent RDD.

如果依赖关系在设计的时候就可以确定, 而不需要考虑父 RDD 分区中的记录, 并且如果父 RDD 中的每个分区最多只有一个子分区, 这样的依赖就叫窄依赖

**父 RDD 的每个分区最多被一个 RDD 的分区使用**

![image-20210713205224892](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210713205224892-1626180745-0b527b.png)

左边为：OneToOneDependency

右边为：RangeDependency

## 宽依赖

如果 父 RDD 的分区被不止一个子 RDD 的分区依赖, 就是宽依赖.

ShuffleDependency

![image-20210713205418681](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210713205418681-1626180858-178523.png)

## Spark Job

由于 Spark 的懒执行, 在驱动程序调用一个action之前, Spark 应用不会做任何事情.

针对每个 **action**, Spark 调度器就创建一个**执行图**(execution graph)和**启动一个 *Spark job***

每个 job 由多个*stages* 组成, 这些 *stages* 就是实现最终的 RDD 所需的数据转换的步骤. 一个宽依赖划分一个 stage.

每个 *stage* 由多个 *tasks* 来组成, 这些 *tasks* 就表示每个并行计算, 并且会在多个执行器上执行.

![image-20210713214619797](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210713214619797-1626183979-1f4aaf.png)

## DAG(Directed Acyclic Graph) 有向无环图

Spark 的顶层调度层使用 RDD 的依赖为每个 job 创建一个由 stages 组成的 DAG(有向无环图). 在 Spark API 中, 这被称作 DAG 调度器(DAG Scheduler).

我们已经注意到, 有些错误, 比如: 连接集群的错误, 配置参数错误, 启动一个 Spark job 的错误, 这些错误必须处理, 并且都表现为 DAG Scheduler 错误. 这是因为一个 Spark job 的执行是被 DAG 来处理.

DAG 为每个 job 构建一个 stages 组成的图表, 从而确定运行每个 task 的位置, 然后传递这些信息给 TaskSheduler. TaskSheduler 负责在集群中运行任务

##  Jobs

Spark job 处于 Spark 执行层级结构中的最高层. 每个 Spark job 对应一个 action, 每个 action 被 Spark 应用中的驱动所程序调用.

可以把 Action 理解成把数据从 RDD 的数据带到其他存储系统的组件(通常是带到驱动程序所在的位置或者写到稳定的存储系统中)

只要一个 action 被调用, Spark 就不会再向这个 job 增加新的东西.

## Stages

 RDD 的转换是懒执行的, 直到调用一个 action 才开始执行 RDD 的转换.

一个 job 是由调用一个 action 来定义的. 一个 action 可能会包含一个或多个转换( transformation ), Spark 根据宽依赖把 job 分解成 stage.

从整体来看, 一个 stage 可以任务是“计算(task)”的集合, 这些每个“计算”在各自的 Executor 中进行运算, 而不需要同其他的执行器或者驱动进行网络通讯. 换句话说, 当任何两个 workers 之间开始需要网络通讯的时候, 这时候一个新的 stage 就产生了, 例如: shuffle 的时候.

这些创建 stage 边界的依赖称为 *ShuffleDependencies*. shuffle 是由宽依赖所引起的, 比如: sort, groupBy, 因为他们需要在分区中重新分发数据. 那些窄依赖的转换会被分到同一个 stage 中.

**设计程序的时候, 尽量少用 shuffle**.

## Tasks

stage 由 tasks 组成. 在执行层级中, task 是最小的执行单位. 每一个 task 表现为一个本地计算.

一个 stage 中的所有 tasks 会对不同的数据执行相同的代码.(程序代码一样, 只是作用在了不同的数据上)

一个 task 不能被多个执行器来执行, 但是, 每个执行器会动态的分配多个 slots 来执行 tasks, 并且在整个生命周期内会并行的运行多个 task. 每个 stage 的 task 的数量对应着分区的数量, 即每个 Partition 都被分配一个 Task 

在大多数情况下, 每个 stage 的所有 task 在下一个 stage 开启之前必须全部完成.

![image-20210713215504219](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210713215504219-1626184504-2b3559.png)
