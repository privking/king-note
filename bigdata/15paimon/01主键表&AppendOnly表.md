# 主键表&AppendOnly表

## 文件目录结构

![cf60933c659066f1353336aa7d6d9654.png](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731234998-c44a11.png)

![image-20241110182949197](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731234659-d429ac.png)

## 分桶方式

`'bucket' = '<num>'` num = -1 动态分桶

建议每个分桶的数据大小在2 GB左右

分区表 固定分桶时，主键字段必须包含分区字段

分区表 动态分桶时，若主键包含分区字段，需要在堆内存维护索引，确定主键与分桶的对应关系

分区表 动态分桶时，若主键不包含分区字段，需要在rocksdb维护全局索引

**桶更新**

1. 固定桶支持动态调整桶的大小（解桶的缩放）。
2. 动态桶在数据量超过一定限制时会自动创建桶
   1. dynamic-bucket.target-row-num：每个分桶最多存储几条数据。默认值为2000000。
   2. dynamic-bucket.initial-buckets：初始的分桶数。如果不设置，初始将会创建等同于writer算子并发数的分桶



## AppendOnly

创建Paimon表时没有指定主键（Primary Key），则该表就是Paimon Append Only表

**Scalable**

定义 bucket 为 -1，且没有主键的分区表

没有桶(都写到bucket-0)的概念，无需考虑数据顺序、无需对数据进行hash partitioning，多个并发可以同时写同一个分区，Scalable 表写入速度更快

**Queue**

默认情况下，Paimon将根据每条数据所有列的取值，确定该数据属于哪个分桶（bucket）。也可以在创建Paimon表时，在WITH参数中指定`bucket-key`参数，不同列的名称用英文逗号分隔。

保证***每个分桶中数据的消费顺序与数据写入Paimon表的顺序一致***

如果表参数中设置了`'scan.plan-sort-partition' = 'true'`，则分区值更小的数据会首先产出，否则创建时间早的分区先产出

对于两条来自相同分区的相同分桶的数据，先写入Paimon表的数据会首先产出

对于两条来自相同分区但不同分桶的数据，由于不同分桶可能被不同的Flink作业并发处理，因此不保证两条数据的消费顺序



### Example

```sql
SET 'execution.runtime-mode' = 'streaming';
-- NONE、AUTO、FORCE
-- 由于分布式系统中的 shuffle 会造成 Changelog 数据的乱序，所以 sink 接收到的数据可能在全局的 upsert 中乱序，所以要在 upsert sink 之前添加一个 upsert 物化算子
SET 'table.exec.sink.upsert-materialize'='NONE';
-- 设置检查点的间隔为1000ms
SET 'execution.checkpointing.interval'=10000;
set parallelism.default=1;
-- table changelog tableau
SET 'sql-client.execution.result-mode' = 'changelog';
CREATE TABLE if not exists datagen1 (
  `id` Int PRIMARY KEY NOT ENFORCED,
  `name` String,
  `age` Int
) with (
'connector' = 'datagen',
'fields.id.kind' = 'random',
'fields.id.min' = '1',
'fields.id.max' = '100',
'fields.name.length' = '10',
'fields.age.min' = '18',
'fields.age.max' = '60',
'rows-per-second' = '3'
);


CREATE CATALOG paimon WITH (
'type' = 'paimon',
'warehouse' = 'hdfs://node1:8020/paimon/lakehouse'
);


USE CATALOG paimon;
create database if not exists test;

-- 固定分桶表主键必须包含分区键
CREATE TABLE if not exists paimon.test.bucket2 (
  id bigint,
  name String,
  age Int,
  dt string,
  PRIMARY KEY (id,dt) NOT ENFORCED
) PARTITIONED BY (dt) with  (
 'bucket' = '2',
 'file.format'='avro',
 'sink.parallelism' = '2' 
);

-- 多作业同时写入一张表
CREATE TABLE if not exists paimon.test.bucket2 (
  id bigint,
  name String,
  age Int,
  dt string,
  PRIMARY KEY (id,dt) NOT ENFORCED
) PARTITIONED BY (dt) with  (
 'bucket' = '2',
 'file.format'='avro',
 'sink.parallelism' = '2',
  'write-only'='true'  -- 不能同时compaction
);

-- 跨分区动态表
-- index 维护在hadoop
CREATE TABLE if not exists paimon.test.bucket2 (
  id bigint,
  name String,
  age Int,
  dt string,
  PRIMARY KEY (id,dt) NOT ENFORCED
) PARTITIONED BY (dt) with  (
 'bucket' = '-1', -- 分桶必须是-1
 'file.format'='avro',
 'sink.parallelism' = '2' 
);

-- 非跨分区动态表
-- index 维护在rocksdb
CREATE TABLE if not exists paimon.test.bucket2 (
  id bigint,
  name String,
  age Int,
  dt string,
  PRIMARY KEY (id) NOT ENFORCED
) PARTITIONED BY (dt) with  (
 'bucket' = '-1', -- 分桶必须是-1
 'file.format'='avro',
 'sink.parallelism' = '2' 
);

-- Scalable 表
CREATE TABLE if not exists paimon.test.bucket2(
  id bigint,
  name String,
  age Int,
  dt string
) PARTITIONED BY (dt) with  (
 'bucket' = '-1'
);


-- Queue表
CREATE TABLE if not exists paimon.test.bucket2 (
  id bigint,
  name String,
  age Int,
  dt string
)  with  (
 'bucket' = '5',
 'bucket-key' = 'id'
);


```

## Changelog producer

主要目的是为了在 Paimon 表上产生流读的 changelog是减小压力, 如果只是批读的表是可以不用设置 Chaneglog producer 的

### None

默认就是 none, 这种模式下在 Paimon 不会额外存储数据。

但是在下游的流任务的状态中, 其实是有一份全量表的额外存储的开销的

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731328826-9eaecc.png)

### Input

指定`'changelog-producer' = 'input'`

依赖他们的输入作为完整更新日志的来源。所有输入记录将保存在单独的changelog中

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731329157-27c6dc.png)

### Lookup

指定`'changelog-producer' = 'lookup'`

性能优化参数

| Option                       | Default   | Type       | Description                                                  |
| :--------------------------- | :-------- | :--------- | :----------------------------------------------------------- |
| lookup.cache-file-retention  | 1 h       | Duration   | The cached files retention time for lookup. After the file expires, if there is a need for access, it will be re-read from the DFS to build an index on the local disk. |
| lookup.cache-max-disk-size   | unlimited | MemorySize | Max disk size for lookup cache, you can use this option to limit the use of local disks. |
| lookup.cache-max-memory-size | 256 mb    | MemorySize | Max memory size for lookup cache.                            |

Paimon主键表在Flink作业每次创建检查点（checkpoint）时触发小文件合并（compaction），并利用小文件合并的结果产生完整的变更数据。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731329362-d60af7.jpeg)

### full-compaction

指定`'changelog-producer' = 'full-compaction'`

指定`full-compaction.delta-commits` 增量提交（检查点）后将不断触发完全压缩

如果觉得lookup的资源消耗太大，可以考虑使用full-compaction changelogproducer，它可以解耦数据写入和changelog生成，更适合延迟较高的场景（比如10分钟）。

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731329541-98fce6.png)



## Merge-Engine

