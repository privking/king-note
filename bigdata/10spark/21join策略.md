# Join策略

Spark提供了5种JOIN机制来执行具体的JOIN操作

- Shuffle Hash Join
- Broadcast Hash Join
- Sort Merge Join
- Cartesian Join
- Broadcast Nested Loop Join

## Shuffle Hash Join

当要JOIN的表数据量比较大时，可以选择Shuffle Hash Join。这样可以将大表进行**按照JOIN的key进行重分区**，保证每个相同的JOIN key都发送到同一个分区中

Shuffle Hash Join的基本步骤主要有以下两点：

- 首先，对于两张参与JOIN的表，分别按照join key进行重分区，该过程会涉及Shuffle，其目的是将相同join key的数据发送到同一个分区，方便分区内进行join。
- 其次，对于每个Shuffle之后的分区，会将小表的分区数据构建成一个Hash table，然后根据join key与大表的分区数据记录进行匹配

#### 条件与特点

- 仅支持等值连接，**join key不需要排序**
- 支持除了全外连接(full outer joins)之外的所有join类型
- 需要对小表构建Hash map，属于内存密集型的操作，如果构建Hash表的一侧数据比较大，可能会造成OOM
- 将参数*spark.sql.join.prefersortmergeJoin (default true)*置为false
- **一定程度上减少了driver广播一侧表的压力，也减少了executor端取整张被广播表的内存消耗,因为被广播的表首先被collect到driver段，然后被冗余分发到每个executor上**

![20181003185922498](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/20181003185922498-1635869260-60195e.png)

## Broadcast Hash Join

也称之为**Map端JOIN**。当有一张表较小时，我们通常选择Broadcast Hash Join，这样可以避免Shuffle带来的开销，从而提高性能。

在进行 Broadcast Join 之前，Spark 需要把需要广播的表从 Executor 端的数据先发送到 Driver 端，然后 Driver 端再把数据广播到 Executor 端。如果我们需要广播的数据比较多，**会造成 Driver 端出现 OOM**。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/broadcastjoin-1635869727-1ad2d0.png)

Broadcast Hash Join主要包括两个阶段：

- **Broadcast**阶段 ：小表被缓存在executor中
- **Hash Join**阶段：在每个 executor中执行Hash Join

#### 条件与特点

- 仅支持等值连接，join key不需要排序
- 支持除了全外连接(full outer joins)之外的所有join类型
- Broadcast Hash Join相比其他的JOIN机制而言，效率更高。但是，Broadcast Hash Join属于网络密集型的操作(数据冗余传输)，除此之外，需要在Driver端缓存数据，所以当小表的数据量较大时，会出现OOM的情况
- 被广播的小表的数据量要小于**spark.sql.autoBroadcastJoinThreshold**值，默认是10MB(10485760)
- 被广播表的大小阈值不能超过8GB
- **executor存储小表的全部数据**，一定程度上牺牲了空间

## Sort Merge Join

该JOIN机制是Spark默认的，可以通过参数**spark.sql.join.preferSortMergeJoin**进行配置，默认是true，即优先使用Sort Merge Join。一般在两张大表进行JOIN时，使用该方式。Sort Merge Join可以减少集群中的数据传输，该方式不会先加载所有数据的到内存，然后进行hashjoin，但是在JOIN之前需要对join key进行排序

等值连接时:因为两个序列都是有序的，从头遍历，碰到key相同的就输出；如果不同，左边小就继续取左边，反之取右边(即用即取即丢)

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/sortmergejoin-1635870127-8a5dbc.png)

Sort Merge Join主要包括三个阶段：

- **Shuffle Phase** : 两张大表根据Join key进行Shuffle重分区
- **Sort Phase**: 每个分区内的数据进行排序
- **Merge Phase**: 对来自不同表的排序好的分区数据进行JOIN，通过遍历元素，连接具有相同Join key值的行来合并数据集

#### 条件与特点

- 仅支持等值连接
- 支持所有join类型
- Join Keys是排序的
- 参数**spark.sql.join.prefersortmergeJoin (默认true)**设定为true

## Cartesian Join

如果 Spark 中两张参与 Join 的表没指定join key（ON 条件）那么会产生 Cartesian product join，这个 Join 得到的结果其实就是两张行数的乘积

#### 条件与特点

- 仅支持内连接
- 支持等值和不等值连接
- 开启参数spark.sql.crossJoin.enabled=true

### Broadcast Nested Loop Join

该方式是在没有合适的JOIN机制可供选择时，最终会选择该种join策略。优先级为：*Broadcast Hash Join > Sort Merge Join > Shuffle Hash Join > cartesian Join > Broadcast Nested Loop Join*.

#### 条件与特点

- 支持等值和非等值连接
- 支持所有的JOIN类型，主要优化点如下：
  - 当右外连接时要广播左表
  - 当左外连接时要广播右表
  - 当内连接时，要广播左右两张表

