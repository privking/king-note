# HDFS
## hadoop中有3个核心组件：
* 分布式文件系统：HDFS —— 实现将文件分布式存储在很多的服务器上
* 分布式运算编程框架：MAPREDUCE —— 实现在很多机器上分布式并行运算
* 分布式资源调度平台：YARN —— 帮用户调度大量的mapreduce程序，并合理分配运算资源

## hdfs整体运行机制
* hdfs：分布式文件系统
* hdfs有着文件系统共同的特征：
    * 有目录结构，顶层目录是：  /
    * 系统中存放的就是文件
    * 系统可以提供对文件的：创建、删除、修改、查看、移动等功能
* hdfs跟普通的单机文件系统有区别：
    * 单机文件系统中存放的文件，是在一台机器的操作系统中
    hdfs的文件系统会横跨N多的机器
    * 单机文件系统中存放的文件，是在一台机器的磁盘上
    * hdfs文件系统中存放的文件，是落在n多机器的本地单机文件系统中（hdfs是一个基于==linux==本地文件系统之上的文件系统）
* hdfs的工作机制
    * 客户把一个文件存入hdfs，其实hdfs会把这个文件切块后，分散存储在N台linux机器系统中（负责存储文件块的角色：==data node==）<准确来说：切块的行为是由客户端决定的>
    * 一旦文件被切块存储，那么，hdfs中就必须有一个机制，来记录用户的每一个文件的切块信息，及每一块的具体存储机器（负责记录块信息的角色是：==name node==）
    * 为了保证数据的安全性，hdfs可以将每一个文件块在集群中存放多个副本（到底存几个副本，是由当时存入该文件的客户端指定的）

## hdfs特性
- 文件权限和身份验证。
- Rack awareness:在调度任务和分配存储时考虑节点的物理位置。
- Safemode:维护管理模式。
- fsck:诊断文件系统运行状况的实用程序，查找丢失的文件或块。
- fetchdt:获取委托令牌并将其存储在本地系统的文件中的实用程序。
- Rebalancer:当数据不均匀地分布在datanode上时，用来平衡集群的工具。
- Upgrade and rollback:软件升级后，如果出现意外问题，可以回滚到升级前的HDFS状态。
- Secondary NameNode:定期检查名称空间，帮助将包含HDFS修改日志的文件的大小保持在NameNode的某些限制内。
- Web Interface：NameNode和DataNode各自运行一个内部web服务器，以显示关于集群当前状态的基本信息。
- Import Checkpoint：如果丢失了映像和编辑文件的所有其他副本，则可以将最新的检查点导入到NameNode


## NameNode 概述
* NameNode是hdfs的核心
* NameNode也成为master
* Namenode仅储存HDFS元数据：文件系统中的所有文件的目录树，文件的分块大小，权限等等，并跟踪整个集群文件中的文件
* NameNode不存储实际数据或数据集。数据本身存储在DataNode中
* NameNode知道HDFS中任何给定文件的块列表及其位置。使用此信息可以知道如何从块中构建文件
* NameNode并不持久化储存每个文件中各个块所在的DataNode位置信息，这些信息会在系统启动的时候从数据节点重建
* NameNode关闭时，集群无法访问
* NameNode所在的机器通常会配置之大量RAM(把元数据保存在内存中)

## SecondaryNameNode

* 将edites和fsimage合并，减少NameNode启动时间
* edites 操作记录，恢复时一条一条指令读取
* fsimage内存镜像
* 合并时机1 **fs.checkpoint.period** 默认3600秒
* 合并时机2 **fs.checkpoint.size**  默认64MB


## DataNode概述
* DataNode负责将实际数据储存在HDFS中
* DataNode也成为Slave
* NameNode和DataNode将保持不断通信
* DataNode启动时，将自己发布到NameNode并汇报自己持有的块列表
* 但某个NameNode关闭时，不会影响数据和集群的可用性，有备份
* DataNode所在机器通常配置有大量的硬盘空间，因为实际数据储存在DataNode中
* DataNode会定期（dfs.heartbeat.interval配置，默认时3秒）先NameNode发送心跳，如果长期没有收到DateNode发送的心跳信息，NameNode将会认为DataNode失效
* block汇报时间配置（dfs.blockreport.intervalMsec 默认为6小时）