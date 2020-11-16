# Hb ase读写

## ReginServer架构

![image-20201102102843004](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102102843004-1604284130-b35710.png)

- StoreFile
  - 保存实际数据的物理文件，StoreFile以Hfile的形式存储在HDFS上。每个Store会有一个或多个StoreFile（HFile），数据在每个StoreFile中都是有序的。
- MemStore
  - 写缓存，由于HFile中的数据要求是有序的，所以数据是先存储在MemStore中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个**新的HFile**。
- WAL
  - 由于数据要经MemStore排序后才能刷写到HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入MemStore中。所以在系统出现故障的时候，数据可以通过这个**日志文件重建**。
  - 每间隔hbase.regionserver.optionallogflushinterval(默认1s)， HBase会把操作从内存写入WAL。 
  - 一个RegionServer上的所有Region**共享一个WAL实例**。
  - WAL的检查间隔由hbase.regionserver.logroll.period定义，默认值为1小时。检查的内容是把当前WAL中的操作跟实际持久化到HDFS上的操作比较，看哪些操作已经被持久化了，被持久化的操作就会被移动到.oldlogs文件夹内（这个文件夹也是在HDFS上的）。一个WAL实例包含有多个WAL文件。WAL文件的最大数量通过hbase.regionserver.maxlogs（默认是32）参数来定义。
- BlockCache
  - 读缓存，每次查询出的数据会缓存在BlockCache中，方便下次查询

## Hbase写流程

![image-20201102105147095](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102105147095-1604285507-8c08d0.png)

- Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
- 访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
- 与目标Region Server进行通讯
- 将数据顺序写入（追加）到WAL；
- 将数据写入对应的MemStore，数据会在MemStore进行排序；
- 向客户端发送ack；
- 等达到MemStore的刷写时机后，将数据刷写到HFile。



## Hbase读流程

![image-20201102104754706](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102104754706-1604285275-3c5680.png)

- Client先访问zookeeper，获取hbase:meta表位于哪个Region Server
- 访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
- 与目标Region Server进行通讯
- 分别在Block Cache（读缓存），MemStore和Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）。
- 将查询到的数据块（Block，HFile数据存储单元，默认大小为64KB）缓存到Block Cache。
- 将合并后的最终结果返回给客户端。

## MemStore Flush

MemStore存在的意义是在写入HDFS前，将其中的数据整理有序。

每个MemStore都会写成一个StoreFile

MemStore刷写时机：

- 当某个memstore（**hbase.hregion.memstore.block.multiplier（默认值4）**）的大小达到了**hbase.hregion.memstore.flush.size（默认值128M）**，其所在region的所有memstore都会刷写
- 当region server中memstore的总大小达到 **java_heapsize**/ **hbase.regionserver.global.memstore.size（默认值0.4）**/**hbase.regionserver.global.memstore.size.lower.limit（默认值0.95）**
- 到达自动刷写的时间，也会触发memstore flush。自动刷新的时间间隔由该属性进行配置**hbase.regionserver.optionalcacheflushinterval（默认1小时）**

## StoreFile Compaction

由于Hbase依赖HDFS存储，HDFS只支持追加写。所以，当新增一个单元格的时候，HBase在HDFS上新增一条数据。当修改一个单元格的时候，HBase在HDFS又新增一条数据，只是版本号比之前那个大（或者自定义）。当删除一个单元格的时候，HBase还是新增一条数据！只是这条数据没有value，类型为DELETE，也称为墓碑标记（Tombstone）

由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的HFile中，因此查询时需要遍历所有的HFile。为了减少HFile的个数，以及清理掉过期和删除的数据，会进行StoreFile Compaction。

Compaction分为两种，分别是Minor Compaction和Major Compaction。Minor Compaction会将临近的若干个较小的HFile合并成一个较大的HFile，但**不会**清理过期和删除的数据。Major Compaction会将一个Store下的所有的HFile合并成一个大HFile，并且**会**清理掉过期和删除的数据。

![image-20201102110632161](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102110632161-1604286392-d88d84.png)

## Region Split

默认情况下，每个Table起初只有一个Region，随着数据的不断写入，Region会自动进行拆分。刚拆分时，两个子Region都位于当前的Region Server，但处于负载均衡的考虑，HMaster有可能会将某个Region转移给其他的Region Server。

Region Split时机

默认使用IncreasingToUpperBoundRegionSplitPolicy策略切分region, getSizeToCheck()是检查region的大小以判断是否满足切割切割条件。

```java
protected long getSizeToCheck(final int tableRegionsCount) {
    // safety check for 100 to avoid numerical overflow in extreme cases
    return tableRegionsCount == 0 || tableRegionsCount > 100
               ? getDesiredMaxFileSize()
               : Math.min(getDesiredMaxFileSize(),
                          initialSize * tableRegionsCount * tableRegionsCount * tableRegionsCount);
  }
//tableRegionsCount：为当前Region Server中属于该Table的region的个数。
//getDesiredMaxFileSize() 这个值是hbase.hregion.max.filesize参数值，默认为10GB。
//initialSize的初始化比较复杂，由多个参数决定。

```

```java
 protected void configureForRegion(HRegion region) {
    super.configureForRegion(region);
Configuration conf = getConf();
//默认hbase.increasing.policy.initial.size 没有在配置文件中指定
    initialSize = conf.getLong("hbase.increasing.policy.initial.size", -1);
    if (initialSize > 0) {
      return;
}
// 获取用户表中自定义的memstoreFlushSize大小，默认也为128M
    HTableDescriptor desc = region.getTableDesc();
    if (desc != null) {
      initialSize = 2 * desc.getMemStoreFlushSize();
}
// 判断用户指定的memstoreFlushSize是否合法，如果不合法，则为hbase.hregion.memstore.flush.size，默认为128. 
    if (initialSize <= 0) {
      initialSize = 2 * conf.getLong(HConstants.HREGION_MEMSTORE_FLUSH_SIZE,
                                     HTableDescriptor.DEFAULT_MEMSTORE_FLUSH_SIZE);
    }
  }
//第一次split：1^3 * 256 = 256MB 
//第二次split：2^3 * 256 = 2048MB 
//第三次split：3^3 * 256 = 6912MB 
//第四次split：4^3 * 256 = 16384MB > 10GB，因此取较小的值10GB 
//后面每次split的size都是10GB了。
```

