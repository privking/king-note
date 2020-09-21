# hdfs读写流程

## 写流程

![在这里插入图片描述](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/20200304192350363-1600139938-d1fe9f.png)

1. 通过Cilent客户端向远程Namenode发送RPC（远程调用）请求
2. ① Namenode 会检查要创建的文件是否已经存在，创建者是否有权限进行操作，成功则会为文件创建一个记录，否则会让客户端抛出异常；
   ② Namenode允许上传文件。同时把待上传的文件按照块大小（128M一块）进行逻辑切分
3. 客户端请求上传第一个Block
4. Namenode 按照设定的副本数拉取一批可用Datanode节点返回给客户端Datanode通过pipeline互相之间进行通信
5. 客户端向Datanode1请求建立通道，Datanode1通过管道依次向Datanode2，Datanode3建立通道。
6. 当返回应答成功时,客户端开启文件输出流
7. 客户端开始以 pipeline（管道）的形式将 packet 写入所有的 replicas（副本节点） 中。**客户端把 packet 以流的 方式写入第一个 datanode，该 datanode 把该 packet 存储之后，再将其传递给在此 pipeline 中的下一个 datanode，直到最后一个 datanode，这种写数据的方式呈流水线的形式**。传输一个packet后datanode向客户端返回传输成功消息，**同时客户端会有一个ack确认队列，成功收到 datanode 返回的 ack packet 后会从"data queue"移除相应的 packet。**

## 读流程

![在这里插入图片描述](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/20200304194520719-1600140060-dc67b9.png)

1. 客户端调用FileSystem 实例的open 方法，获得这个文件对应的输入流InputStream。
2. 通过RPC 远程调用NameNode ，获得NameNode 中此文件对应的数据块保存位置，包括这个文件的副本的保存位置( 主要是各DataNode的地址) 。
3. 获得输入流之后，客户端调用read 方法读取数据。选择最近的DataNode 建立连接并读取数据。
4. 如果客户端和其中一个DataNode 位于同一机器(比如MapReduce 过程中的mapper 和reducer)，那么就会直接从本地读取数据。
5. 到达数据块末端，关闭与这个DataNode 的连接，然后重新查找下一个数据块。
6. 不断执行第2 - 5 步直到数据全部读完。
7. 客户端调用close ，关闭输入流FS InputStream。