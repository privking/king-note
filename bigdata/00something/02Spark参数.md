# Spark 参数配置

https://spark.apache.org/docs/latest/configuration.html

https://spark.apache.org/docs/latest/sql-performance-tuning.html

| Name                                                         | Default                                                      | Meaning                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
| spark.app.name                                               | None                                                         | 应用名                                                       |
| spark.driver.cores                                           | 1                                                            | driver 核心数                                                |
| spark.driver.memory                                          | 1g                                                           | driver内存                                                   |
| spark.driver.memoryOverhead                                  | driverMemory * `spark.driver.memoryOverheadFactor`, with minimum of 384 | driver预留内存大小                                           |
| spark.driver.memoryOverheadFactor                            | 0.1                                                          | 预留内存占比因子                                             |
| spark.executor.memory                                        | 1g                                                           | executor内存                                                 |
| spark.executor.memoryOverhead                                | executorMemory * `spark.executor.memoryOverheadFactor`, with minimum of 384 | executor预留内存大小                                         |
| spark.executor.memoryOverheadFactor                          | 0.1                                                          | 预留内存占比因子                                             |
| spark.driver.maxResultSize                                   | 1g                                                           | 返回每个分区序列化结果到driver的最大大小，适用于将数据返回driver的算子 |
| spark.reducer.maxSizeInFlight                                | 48m                                                          | shuffle read task的buffer缓冲大小，而这个buffer缓冲决定了每次能够拉取多少数据 |
| spark.reducer.maxReqsInFlight                                | Int.MaxValue                                                 | 限制在任何给定点获取块的远程请求数量，当集群过大是避免负载过高 |
| spark.reducer.maxBlocksInFlightPerAddress                    | Int.MaxValue                                                 | 限制了每个 Reduce 任务从给定主机端口获取的远程块数。当在一次获取中或同时从给定地址请求大量块时，这可能会使服务执行器或节点管理器崩溃。当启用外部 shuffle 时，这对于减少节点管理器上的负载特别有用 |
| spark.shuffle.compress                                       | true                                                         | shuffle压缩 `spark.io.compression.codec`                     |
| spark.shuffle.file.buffer                                    | 32k                                                          | 每个 shuffle 文件输出流的内存缓冲区大小,这些缓冲区可减少创建中间 shuffle 文件时进行的磁盘寻道和系统调用次数 |
| spark.shuffle.unsafe.file.output.buffer                      | 32k                                                          | Unsafe shuffle writer 缓冲区                                 |
| spark.shuffle.spill.diskWriteBufferSize                      | 1024 * 1024                                                  | The buffer size, in bytes, to use when writing the sorted records to an on-disk file. |
| spark.shuffle.io.maxRetries                                  | 3                                                            | 发生io相关异常的时候重试，在长时间gc或网络不稳定的情况下有用 |
| spark.shuffle.io.numConnectionsPerPeer                       | 1                                                            | 主机之间的连接会被重用，以减少大型集群的连接积累。对于硬盘较多而主机较少的集群，这可能会导致并发量不足以饱和所有磁盘 |
| spark.shuffle.io.preferDirectBufs                            | true                                                         | 优先堆外缓冲区                                               |
| spark.shuffle.io.retryWait                                   | 5s                                                           | io重试间隔                                                   |
| spark.shuffle.io.connectionTimeout                           | value of `spark.network.timeout`                             | 连接超时                                                     |
| spark.shuffle.io.connectionCreationTimeout                   | value of `spark.shuffle.io.connectionTimeout`                | 创建连接超时                                                 |
| spark.shuffle.service.enabled                                | false                                                        | ESS                                                          |
| spark.shuffle.maxChunksBeingTransferred                      | Long.MAX_VALUE                                               | 在 shuffle 服务上允许同时传输的最大块数                      |
| spark.shuffle.sort.bypassMergeThreshold                      | 200                                                          | bypassMergeShuffle 启用限定的分区数                          |
| spark.shuffle.spill.compress                                 | true                                                         | shuffle spill 压缩 `spark.io.compression.codec`              |
| spark.shuffle.<br/>minNumPartitionsToHighlyCompress          | 2000                                                         | 当下游 ReduceTask 个数大于某一阈值(默认 2000)，就会将MapStatus进行压缩，所有小于 accurateBlockThreshold（默认100M）的值都会被一个平均值所代替填充,**可能会影响AQE** |
| spark.shuffle.accurateBlockThreshold                         | 100M                                                         |                                                              |
| spark.shuffle.reduceLocality.enabled                         | true                                                         | 是否为reduce任务配置本机计算偏好                             |
| spark.shuffle.mapOutput.minSizeForBroadcast                  | 512k                                                         | executor获取mapstatus的时候当数据量较大时使用广播            |
| spark.shuffle.detectCorrupt                                  | true                                                         | 检测获取到的block中是否有损坏                                |
| spark.shuffle.detectCorrupt.useExtraMemory                   | false                                                        | 如果流被压缩或包装，那么我们可选择将前 maxBytesInFlight/3 个字节解压缩/解包到内存中，以检查该部分数据是否损坏。但即使“detectCorruptUseExtraMemory”配置已关闭，或者如果损坏发生较晚，我们仍会在稍后的流中检测到损坏 |
| spark.shuffle.useOldFetchProtocol                            | false                                                        |                                                              |
| spark.shuffle.readHostLocalDisk                              | true                                                         | 如果启用（并且 spark.shuffle.useOldFetchProtocol 被禁用），则从同一主机上运行的块管理器请求的 shuffle 块将直接从磁盘读取 |
| spark.shuffle.checksum.enabled                               | true                                                         | 是否计算 shuffle 数据的校验和。如果启用，Spark 将计算 map 输出文件内每个分区数据的校验和值，并将这些值存储在磁盘上的校验和文件中。当检测到 shuffle 数据损坏时，Spark 将尝试使用校验和文件来诊断损坏的原因（例如网络问题、磁盘问题等）。 |
| spark.shuffle.checksum.algorithm                             | ADLER32                                                      | ADLER32, CRC32.                                              |
| spark.broadcast.compress                                     | true                                                         | 广播数据压缩 `spark.io.compression.codec`                    |
| spark.checkpoint.compress                                    | false                                                        | `spark.io.compression.codec`                                 |
| spark.io.compression.codec                                   | lz4                                                          | lz4,lzf, snappy, and zstd                                    |
| spark.serializer                                             | org.apache.spark.serializer.JavaSerializer                   |                                                              |
| spark.memory.fraction                                        | 0.6                                                          | 用于执行和存储的内存占总内存的比例                           |
| spark.memory.storageFraction                                 | 0.5                                                          | 执行内存和存储内存的比例                                     |
| spark.memory.offHeap.enabled                                 | false                                                        | 是否开启堆外内存                                             |
| spark.memory.offHeap.size                                    | 0                                                            | 堆外内存大小  byte                                           |
| spark.broadcast.blockSize                                    | 4m                                                           | TorrentBroadcastFactory 中每个块的大小，值过大会影响广播并行度，值过小会影响BlockManager |
| spark.broadcast.checksum                                     | true                                                         | enable checksum for broadcast                                |
| spark.executor.cores                                         | 1                                                            | executor的核心数                                             |
| spark.default.parallelism                                    | join、reduceByKey 和 parallelize 等转换返回                  | RDD默认分区数                                                |
| spark.locality.wait                                          | 3s                                                           | 在放弃并在非本地节点上启动数据本地任务之前，等待多长时间启动该任务。相同的等待时间将用于逐步完成多个本地级别（进程本地、节点本地、机架本地，然后是任何级别）,默认值已经效果较好 |
| spark.scheduler.mode                                         | FIFO                                                         | 同一个context下的调度策略 FAIR、FIFO                         |
| spark.excludeOnFailure.enabled                               | false                                                        | 禁止在失败过多任务的executor上调度任务                       |
| spark.speculation                                            | false                                                        | 推测执行，不同task的执行时间可能不一样，有的task很快就执行完成了，而有的可能执行很长一段时间也没有完成，再启动一个task,完成后将另外一个kill掉，对集群中性能不一致的机器比较适用 |
| spark.task.cpus                                              | 1                                                            | Number of cores to allocate for each task.                   |
| spark.dynamicAllocation.enabled                              | false                                                        | 动态申请资源                                                 |
| spark.dynamicAllocation.executorIdleTimeout                  | 60s                                                          |                                                              |
| spark.dynamicAllocation.cachedExecutorIdleTimeout            | infinity                                                     |                                                              |
| spark.dynamicAllocation.initialExecutors                     | spark.dynamicAllocation.minExecutors                         |                                                              |
| spark.dynamicAllocation.maxExecutors                         | infinity                                                     |                                                              |
| spark.dynamicAllocation.minExecutors                         | 0                                                            |                                                              |
| spark.dynamicAllocation.executorAllocationRatio              | 1                                                            | 默认情况下，动态分配将根据要处理的任务数量请求足够的执行器以最大化并行度。虽然这可以最大限度地减少作业的延迟，但对于小任务，此设置可能会由于执行器分配开销而浪费大量资源 |
| spark.dynamicAllocation.schedulerBacklogTimeout              | 1s                                                           | 如果启用了动态分配，并且待处理任务积压的时间超过此时间，则会请求新的执行器 |
| spark.dynamicAllocation.shuffleTracking.enabled              | true                                                         | 保持有shuffle数据的executor存活                              |
| spark.dynamicAllocation.shuffleTracking.timeout              | infinity                                                     |                                                              |
| spark.sql.adaptive.advisoryPartitionSizeInBytes              | 67108864                                                     | shuffle分区的建议大小，用于合并小分区或切割数据倾斜分区，spark.sql.adaptive.shuffle.targetPostShuffleInputSize 3.0以前的版本为该参数 |
| spark.sql.adaptive.autoBroadcastJoinThreshold                | none（spark.sql.autoBroadcastJoinThreshold）                 | 最大能广播的大小                                             |
| spark.sql.adaptive.localShuffleReader.enabled                | true                                                         | 将 sort-merge join 转换为 broadcast-hash join 之后,尝试使用本地 shuffle reader 来读取 shuffle 数据 |
| spark.sql.adaptive.coalescePartitions.enabled                | true                                                         | 根据spark.sql.adaptive.advisoryPartitionSizeInBytes大小合并相邻的小分区 |
| spark.sql.adaptive.coalescePartitions.initialPartitionNum    | none                                                         | 如果未配置 默认 spark.sql.shuffle.partitions，初始化较高的值，然后再通过连续分许合并解决避免小任务，较高的初始值能是数据分布更加均匀 |
| spark.sql.adaptive.coalescePartitions.minPartitionSize       | 1MB                                                          | 合并后最小的分区大小，其值最多为 spark.sql.adaptive.advisoryPartitionSizeInBytes 的 20%。当分区合并期间忽略目标大小（这是默认情况）时，这很有用 |
| spark.sql.adaptive.coalescePartitions.parallelismFirst       | true                                                         | Spark 在合并连续的 shuffle 分区时会忽略 spark.sql.adaptive.advisoryPartitionSizeInBytes（默认 64MB）指定的目标大小，并且仅遵守 spark.sql.adaptive.coalescePartitions.minPartitionSize（默认 1MB）指定的最小分区大小，以最大化并行度。这是为了避免在启用自适应查询执行时出现性能下降。建议将此配置设置为 false |
| spark.sql.adaptive.enabled                                   | true                                                         | 开启aqe优化                                                  |
| spark.sql.adaptive.maxShuffledHashJoinLocalMapThreshold      | 0                                                            | 配置允许构建本地哈希映射的每个分区的最大大小（以字节为单位）。如果此值不小于 spark.sql.adaptive.advisoryPartitionSizeInBytes 且所有分区大小不大于此配置，则无论 spark.sql.join.preferSortMergeJoin 的值如何，连接选择都会优先使用 shuffled hash join 而不是 sort merge join。 |
| spark.sql.adaptive.skewJoin.enabled                          | true                                                         |                                                              |
| spark.sql.adaptive.skewJoin.skewedPartitionFactor            | 5.0                                                          | 如果分区大小大于此因子乘以中值分区大小，并且大于 spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes，则认为分区倾斜 |
| spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes  | 256MB                                                        | 如果分区大小（以字节为单位）大于此阈值，并且大于 spark.sql.adaptive.skewJoin.skewedPartitionFactor 与中值分区大小的乘积，则认为该分区倾斜。理想情况下，此配置应设置为大于 spark.sql.adaptive.advisoryPartitionSizeInBytes。 |
| spark.sql.adaptive.forceOptimizeSkewedJoin                   | false                                                        | 当为真时，强制启用 OptimizeSkewedJoin，这是一个自适应规则，用于优化倾斜连接以避免落后任务，即使它引入了额外的混洗。 |
| spark.sql.adaptive.optimizeSkewsInRebalancePartitions.enabled | true                                                         | rebalance 时候根据spark.sql.adaptive.advisoryPartitionSizeInBytes合并小分区 |
| spark.sql.adaptive.rebalancePartitionsSmallPartitionFactor   | 0.2                                                          | 如果分区的大小小于此因子乘以 spark.sql.adaptive.advisoryPartitionSizeInBytes，则将在拆分期间合并分区。 |
| spark.sql.autoBroadcastJoinThreshold                         | 10MB                                                         |                                                              |
| spark.sql.broadcastTimeout                                   | 300                                                          | 广播加入时的广播等待时间超时（秒）。                         |
| spark.sql.cbo.enabled                                        | false                                                        |                                                              |
| spark.sql.cbo.joinReorder.dp.star.filter                     | false                                                        | 用于控制星型查询（大表join多个小表）在动态规划连接重排序过程中的过滤器应用的配置选项。启用该选项可以优化连接顺序，提升查询性能 |
| spark.sql.cbo.joinReorder.dp.threshold                       | 12                                                           | 动态规划算法中允许的最大连接节点数。                         |
| spark.sql.cbo.joinReorder.enabled                            | false                                                        |                                                              |
| spark.sql.cbo.planStats.enabled                              | false                                                        | 当为真时，逻辑计划将从目录中获取行数和列统计信息。           |
| spark.sql.cbo.starSchemaDetection                            | false                                                        | 根据星型模式检测启用连接重新排序                             |
| spark.sql.files.maxPartitionBytes                            | 128MB                                                        | 读取文件时打包到单个分区的最大字节数。此配置仅在使用基于文件的源（例如 Parquet、JSON 和 ORC）时有效。 |
| spark.sql.files.maxPartitionNum                              | none                                                         | 建议（不保证）的最大分割文件分区数。如果设置了该值，当初始分区数超过该值时，Spark 将重新缩放每个分区以使分区数接近该值。此配置仅在使用基于文件的源（例如 Parquet、JSON 和 ORC）时有效。 |
| spark.sql.files.maxRecordsPerFile                            | 0                                                            | 写入单个文件的最大记录数。如果此值为零或负数，则没有限制。   |
| spark.sql.files.minPartitionNum                              | none                                                         | 建议（不保证）的最小文件分区数。如果未设置，则默认值为 spark.sql.leafNodeDefaultParallelism。此配置仅在使用基于文件的源（例如 Parquet、JSON 和 ORC）时有效。 |
| spark.sql.optimizer.dynamicPartitionPruning.enabled          | true                                                         |                                                              |
| spark.sql.optimizer.runtime.bloomFilter.applicationSideScanSizeThreshold | 10GB                                                         | Bloom 过滤器应用端的聚合扫描字节大小需要超过此值才能注入 Bloom 过滤器 |
| spark.sql.optimizer.runtime.bloomFilter.creationSideThreshold | 10MB                                                         | 布隆过滤器创建端计划的大小阈值。估计大小需要低于此值才能尝试注入布隆过滤器。 |
| spark.sql.optimizer.runtime.bloomFilter.enabled              | true                                                         |                                                              |
| spark.sql.optimizer.runtime.bloomFilter.expectedNumItems     | 1000000                                                      | 运行时布隆过滤器的默认预期items                              |
| spark.sql.optimizer.runtime.bloomFilter.maxNumBits           | 67108864                                                     | 布隆过滤器使用的最大位数                                     |
| spark.sql.optimizer.runtime.bloomFilter.maxNumItems          | 4000000                                                      | 布隆过滤器允许的最大预期items                                |
| spark.sql.optimizer.runtime.bloomFilter.numBits              | 8388608                                                      | 布隆过滤器使用的默认位数                                     |
| spark.sql.optimizer.runtime.rowLevelOperationGroupFilter.enabled | true                                                         |                                                              |
| spark.sql.optimizer.runtimeFilter.number.threshold           | 10                                                           | 单个查询中注入的运行时过滤器（非 DPP）总数。这是为了防止驱动程序因过多的布隆过滤器而导致 OOM。 |
| spark.sql.optimizer.runtimeFilter.semiJoinReduction.enabled  | false                                                        |                                                              |
| spark.sql.parquet.aggregatePushdown                          | false                                                        | 如果为 true，聚合将被推送到 Parquet 进行优化。支持 MIN、MAX 和 COUNT 作为聚合表达式。对于 MIN/MAX，支持布尔值、整数、浮点数和日期类型。对于 COUNT，支持所有数据类型。如果任何 Parquet 文件页脚缺少统计信息，则会抛出异常。 |
| spark.sql.parquet.filterPushdown                             | true                                                         | 启用 Parquet 过滤器下推优化                                  |
| spark.sql.shuffle.partitions                                 | 200                                                          | spark.default.parallelism 配置负责控制默认RDD的partithion数，spark.sql.shuffle.partitions 执行sql或sql类算子时shuffle分区数**spark.default.parallelism 主要用于控制 RDD 操作的默认并行度级别，而不是 Spark SQL，所以对于 Spark SQL 并不生效** |
| spark.sql.shuffledHashJoinFactor                             | 3                                                            | 如果小端的数据大小乘以该因子后仍然小于大端，则可以选择 Shuffle Hash Join。 |
| spark.sql.statistics.fallBackToHdfs                          | false                                                        | 如果为 true，则如果无法从表元数据中获得表统计信息，它将回退到 HDFS。这对于确定表是否足够小以使用广播连接非常有用。此标志仅对非分区 Hive 表有效。对于非分区数据源表，如果无法获得表统计信息，它将自动重新计算。对于分区数据源和分区 Hive 表，如果无法获得表统计信息，则为“spark.sql.defaultSizeInBytes”。 |
| spark.sql.statistics.histogram.enabled                       | false                                                        | 在计算列统计信息时生成直方图。直方图可以提供更好的估算精度   |
| spark.sql.statistics.size.autoUpdate.enabled                 | false                                                        | 一旦表的数据发生变化，启用表大小的自动更新。如果表的文件总数非常大，这可能会很昂贵并减慢数据更改命令的速度 |
| spark.sql.ui.explainMode                                     | formatted                                                    | 配置 Spark SQL UI 中使用的查询解释模式。值可以是 'simple'、'extended'、'codegen'、'cost' 或 'formatted'。默认值为 'formatted'。 |