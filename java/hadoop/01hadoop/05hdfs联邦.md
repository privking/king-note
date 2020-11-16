# hdfs联邦

## 单节点局限性

1. NameSpace（命名空间的限制）：由于Namenode在内存中存储所有的元数据（metadata）。NN在管理大规模的命名空间时，单个Namenode所能存储的对象（文件+块）数目受到Namenode所在JVM的堆【内存大小的限制】。
2. 性能的瓶颈：由于是单个Namenode的HDFS架构，因此整个HDFS文件系统的吞吐量受限于单个NameNode的吞吐量。
3. 隔离问题:由于仅有一个Namenode，无法隔离各个程序，因此HDFS上的一个实验程序很可能影响整个HDFS上运行的程序。
4. 集群的可用性:在只有一个Namenode的HDFS中，此Namenode的宕机无疑会导致整个集群的不可用。（低可用性）

## 联邦机制

Federation是指HDFS集群可使用多个独立的NameSpace(NameNode节点管理)来满足HDFS命名空间的水平扩展
这些NameNode分别管理一部分数据，且共享所有DataNode的存储资源。

NameSpace之间在逻辑上是完全相互独立的(即任意两个NameSpace可以有完全相同的文件名)。在物理上可以完全独立(每个NameNode节点管理不同的DataNode)也可以有联系(共享存储节点DataNode)。一个NameNode节点只能管理一个Namespace

## 联邦机制解决的问题

1. HDFS集群扩展性。每个NameNode分管一部分namespace，相当于namenode是一个分布式的。
2. 性能更高效。多个NameNode同时对外提供服务，提供更高的读写吞吐率。
3. 良好的隔离性。用户可根据需要将不同业务数据交由不同NameNode管理，这样不同业务之间影响很小。
4. Federation良好的向后兼容性，已有的单Namenode的部署配置不需要任何改变就可以继续工作。

## 联邦机制不足之处

HDFS Federation并没有完全解决单点故障问题。虽然namenode/namespace存在多个，但是从单个namenode/namespace看，仍然存在单点故障。因此 Federation中每个namenode配置成HA高可用集群，以便主namenode挂掉一下，用于快速恢复服务。

## 架构

![这里写图片描述](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/20180327203932360-1600138606-6fdb6f.gif)