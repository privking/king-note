## 基础特性

### 提交策略

Paimon写入器使用两相提交协议原子向表提交一批记录。每次提交在提交时最多生成两个快照。这取决于增量写入和压缩策略。如果只执行增量写入而不触发压缩操作，则只会创建增量快照。如果触发压缩操作，将创建增量快照和压缩快照。

对于任何两个同时修改表的写入者，只要他们不修改相同的分区，他们的提交就可以并行进行。如果他们修改了相同的分区，只保证快照隔离。

### 并发控制

Paimon支持多个并发写入作业的乐观并发。

**快照冲突**

对于hdfs,使用rename的原子特性保证

对于对象存储，rename无法保证原子性，需要配置hive或jdbc元数据并允许 `lock.enabled`

**文件冲突**

当Paimon提交文件删除（这只是逻辑删除）时，它会检查与最新快照的冲突。如果存在冲突（这意味着文件已在逻辑上被删除），它不能再在此提交节点上继续，因此它只能故意触发故障转移以重新启动





## 主键表&AppendOnly表

### 分桶方式

`'bucket' = '<num>'` num = -1 动态分桶

建议每个分桶的数据大小在2 GB左右

分区表 固定分桶时，主键字段必须包含分区字段

分区表 动态分桶时，若主键包含分区字段，需要在堆内存维护索引，确定主键与分桶的对应关系

分区表 动态分桶时，若主键不包含分区字段，需要在rocksdb维护全局索引

**桶更新**

1. 固定桶支持动态调整桶的大小（桶的缩放）。
2. 动态桶在数据量超过一定限制时会自动创建桶
   1. dynamic-bucket.target-row-num：每个分桶最多存储几条数据。默认值为2000000。
   2. dynamic-bucket.initial-buckets：初始的分桶数。如果不设置，初始将会创建等同于writer算子并发数的分桶



### AppendOnly

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

-- 动态表,主键包含分区
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

-- 动态表，主键不包含分区
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

## Index

### Dynamic Bucket Index（Table Index）

分区字段包含部分主键，并且桶的数量是-1

动态桶索引用于存储主键的散列值和桶之间的对应关系

只存储主键的hash值

## Deletion Vectors(Table Index)

删除文件用于存储每个数据文件的已删除记录位置。每个桶都有一个主键的删除文件（索引文件，index目录下）

MOW模式

### File Index

Define `'file-index.bloom-filter.columns'`.  

Define `'file-index.bitmap.columns'`.

定义`file-index.${index_type}.columns`，Paimon将为每个文件创建相应的索引文件。如果索引文件太小，它将直接存储在清单或数据文件的目录中。每个数据文件对应一个索引文件，该文件具有单独的文件定义，可以包含具有多列的不同类型的索引。





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

在Flink SQL TableConfig中始终将t`able.exec.sink.upsert-materialize`设置为NONE，sink upsert-materialize可能会导致奇怪的行为。当输入不按顺序时，建议使用序列字段来纠正混乱。



### deduplicate

`'merge-engine' = 'deduplicate'`

默认值

对于多条相同主键的数据，会保留最新的一条数据

### first-row

`'merge-engine' = 'first-row'` 

只会保留相同主键数据中的第一条

支持的changelog producer 有 None和Lookup

如果下游要流式消费数据，需要设置为Lookup

无法处理delete 和 update before ,可以设置`'first-row.ignore-delete' = 'true'`

### aggregation

`'merge-engine' = 'aggregation'` 

有相同主键的多条数据，主键表将会根据指定的聚合函数进行聚合。对于不属于主键的每一列，都需要通过`fields.<field-name>.aggregate-function`指定一个聚合函数，否则该列将默认使用last_non_null_value聚合函数。

### partial-update

`'merge-engine' = 'partial-update'`

可以通过多条消息对数据进行逐步更新，并最终得到完整的数据。即具有相同主键的新数据将会覆盖原来的数据，但值为null的列不会进行覆盖。

如果下游需要流式消费partial-update的结果，`changelog-producer`参数设为input、lookup或full-compaction。

partial-update 无法处理 delete 与 update_before 消息。需要设置`'partial-update.ignore-delete' = 'true'` 以忽略这两类消息。

**Sequence Group**

可以解决多流更新时乱序的问题

```sql
CREATE TABLE t
(
    k   INT,
    a   INT,
    b   INT,
    g_1 INT,
    c   INT,
    d   INT,
    g_2 INT,
    PRIMARY KEY (k) NOT ENFORCED
) WITH (
      'merge-engine' = 'partial-update',
      'fields.g_1.sequence-group' = 'a,b',
      'fields.g_2.sequence-group' = 'c,d'
      );

INSERT INTO t
VALUES (1, 1, 1, 1, 1, 1, 1);

-- g_2 is null, c, d should not be updated
INSERT INTO t
VALUES (1, 2, 2, 2, 2, 2, CAST(NULL AS INT));

SELECT *
FROM t;
-- output 1, 2, 2, 2, 1, 1, 1

-- g_1 is smaller, a, b should not be updated
INSERT INTO t
VALUES (1, 3, 3, 1, 3, 3, 3);

SELECT *
FROM t; -- output 1, 2, 2, 2, 3, 3, 3
```

`fields.<field-name>.sequence-group`，有效的比较数据类型包括：DECIMAL、TINYINT、SMALLINT、INTEGER、BIGINT、FLOAT、DOUBLE、DATE、TIME、TIMESTAMP 和 TIMESTAMP_LTZ。

**支持部分聚合**

```sql
CREATE TABLE t
(
    k INT,
    a INT,
    b INT,
    c INT,
    d INT,
    PRIMARY KEY (k) NOT ENFORCED
) WITH (
      'merge-engine' = 'partial-update',
      'fields.a.sequence-group' = 'b',
      'fields.b.aggregate-function' = 'first_value',
      'fields.c.sequence-group' = 'd',
      'fields.d.aggregate-function' = 'sum'
      );
```

## 文件布局

**数据文件**

Paimon支持使用parquet（默认）、orc和avro作为数据文件的格式。

orc,parquet 列式存储

avro 行式存储





![image-20241110182949197](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731940395-9e6004.png)

![image-20241118223430355](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1731940470-652bf2.png)

### schema

```SQL
{
  "version" : 2,
  "id" : 0,
  "fields" : [ {
    "id" : 0,
    "name" : "id",
    "type" : "INT NOT NULL"
  }, {
    "id" : 1,
    "name" : "name",
    "type" : "STRING"
  }, {
    "id" : 2,
    "name" : "age",
    "type" : "INT"
  }, {
    "id" : 3,
    "name" : "dt",
    "type" : "STRING NOT NULL"
  } ],
  "highestFieldId" : 3,
  "partitionKeys" : [ "dt" ],
  "primaryKeys" : [ "id", "dt" ],
  "options" : {
    "bucket" : "1",
    "changelog-producer" : "input"
  },
  "comment" : "",
  "timeMillis" : 1728655009691
}
```

### snapshot

```JSON
{
  "version" : 3,
  "id" : 1,
  "schemaId" : 0,
  "baseManifestList" : "manifest-list-c935e5b9-5701-4aa9-951a-61ceef8ffd23-0",
  "deltaManifestList" : "manifest-list-c935e5b9-5701-4aa9-951a-61ceef8ffd23-1",
  "changelogManifestList" : "manifest-list-c935e5b9-5701-4aa9-951a-61ceef8ffd23-2",
  "commitUser" : "3bd48472-3f17-4753-a663-fc7e3b58015a",
  "commitIdentifier" : 1,
  "commitKind" : "APPEND",
  "timeMillis" : 1728655078994,
  "logOffsets" : { },
  "totalRecordCount" : 39,
  "deltaRecordCount" : 39,
  "changelogRecordCount" : 50,
  "watermark" : -9223372036854775808
}
```

### baseManifestList

```json
{
  "org.apache.paimon.avro.generated.record" : {
    "_VERSION" : 2,
    "_FILE_NAME" : "manifest-e473d040-9eb8-4cdc-b7d6-ccc361d158aa-0",
    "_FILE_SIZE" : 1881,
    "_NUM_ADDED_FILES" : 1,
    "_NUM_DELETED_FILES" : 0,
    "_PARTITION_STATS" : {
      "org.apache.paimon.avro.generated.record__PARTITION_STATS" : {
        "_MIN_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u0000\u0010\u0000\u0000\u000020241011",
        "_MAX_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u0000\u0010\u0000\u0000\u000020241011",
        "_NULL_COUNTS" : {
          "array" : [ {
            "long" : 0
          } ]
        }
      }
    },
    "_SCHEMA_ID" : 0
  }
}
```

### deltaManifestList

```json
{
  "org.apache.paimon.avro.generated.record" : {
    "_VERSION" : 2,
    "_FILE_NAME" : "manifest-e473d040-9eb8-4cdc-b7d6-ccc361d158aa-2",
    "_FILE_SIZE" : 1883,
    "_NUM_ADDED_FILES" : 1,
    "_NUM_DELETED_FILES" : 0,
    "_PARTITION_STATS" : {
      "org.apache.paimon.avro.generated.record__PARTITION_STATS" : {
        "_MIN_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u0000\u0010\u0000\u0000\u000020241011",
        "_MAX_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u0000\u0010\u0000\u0000\u000020241011",
        "_NULL_COUNTS" : {
          "array" : [ {
            "long" : 0
          } ]
        }
      }
    },
    "_SCHEMA_ID" : 0
  }
}
```

### **manifest**

```json
{
  "org.apache.paimon.avro.generated.record" : {
    "_VERSION" : 2,
    "_KIND" : 0,
    "_PARTITION" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u0000\u0010\u0000\u0000\u000020241011",
    "_BUCKET" : 0,
    "_TOTAL_BUCKETS" : 1,
    "_FILE" : {
      "org.apache.paimon.avro.generated.record__FILE" : {
        "_FILE_NAME" : "changelog-220251f9-61cc-4f63-8c3a-4a7843bc913e-2.orc",
        "_FILE_SIZE" : 1677,
        "_ROW_COUNT" : 60,
        "_MIN_KEY" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0002\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
        "_MAX_KEY" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000c\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
        "_KEY_STATS" : {
          "org.apache.paimon.avro.generated.record__FILE__KEY_STATS" : {
            "_MIN_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0002\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
            "_MAX_VALUES" : "\u0000\u0000\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000c\u0000\u0000\u0000\u0000\u0000\u0000\u0000",
            "_NULL_COUNTS" : {
              "array" : [ {
                "long" : 0
              } ]
            }
          }
        },
        "_VALUE_STATS" : {
          "org.apache.paimon.avro.generated.record__FILE__VALUE_STATS" : {
            "_MIN_VALUES" : "\u0000\u0000\u0000\u0004\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0002\u0000\u0000\u0000\u0000\u0000\u0000\u0000\n\u0000\u0000\u0000(\u0000\u0000\u0000\u0012\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u00008\u0000\u0000\u000005d7db5a7c\u0000\u0000\u0000\u0000\u0000\u000020241011",
            "_MAX_VALUES" : "\u0000\u0000\u0000\u0004\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000c\u0000\u0000\u0000\u0000\u0000\u0000\u0000\n\u0000\u0000\u0000(\u0000\u0000\u0000;\u0000\u0000\u0000\u0000\u0000\u0000\u0000\b\u0000\u0000\u00008\u0000\u0000\u0000fe983096ed\u0000\u0000\u0000\u0000\u0000\u000020241011",
            "_NULL_COUNTS" : {
              "array" : [ {
                "long" : 0
              }, {
                "long" : 0
              }, {
                "long" : 0
              }, {
                "long" : 0
              } ]
            }
          }
        },
        "_MIN_SEQUENCE_NUMBER" : 50,
        "_MAX_SEQUENCE_NUMBER" : 109,
        "_SCHEMA_ID" : 0,
        "_LEVEL" : 0,
        "_EXTRA_FILES" : [ ],
        "_CREATION_TIME" : {
          "long" : 1728683938172
        },
        "_DELETE_ROW_COUNT" : {
          "long" : 0
        },
        "_EMBEDDED_FILE_INDEX" : null
      }
    }
  }
}

```



![无标题-2024-11-25-2300](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1732728008-cb12f7.png)



## 快照管理

**快照清理的是快照以及快照指向的数据文件，并不会把我们的表的数据清理掉，changelog文件会清理掉，data文件会进行合并（所以相当于部分情况下历史数据增删除修改会不准确）**

1. 离线（查询不到数据）：批量查询快照数据查找不到文件。比如表很大查询10分钟，但是10分钟前的快照过期了，此时批量查询读取历史快照查询不到数据
2. 实时（启动报错）：实时消费依赖于changelog，如果下游消费者作了savepoint停止服务了，当任务重启的时候，从savepoint、chenkpoint点继续消费快照，快照过期了，则启动报错

| 参数                      | 说明                       | 数据类型 | 默认值     |
| ------------------------- | -------------------------- | -------- | ---------- |
| snapshot.num-retained.min | 至少保留几个快照文件。     | Integer  | 10         |
| snapshot.num-retained.max | 至多保留几个快照文件。     | Integer  | 2147483647 |
| snapshot.time-retained    | 一个快照文件最长保留多久。 | Duration | 1h         |

## 设置分区过期时间

**只有包含分区过期事件的快照文件也过期了，分区中的数据文件才会被真正删除**

| 参数                          | 说明                               | 备注                                                         |
| ----------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| partition.expiration-time     | 分区的过期时间。                   | 参数值为时间长度，例如12h、7d等。                            |
| partition.timestamp-pattern   | 将分区值转换为时间字符串的格式串。 | 在该格式串中，分区列由美元符号（$）加上列名表示。            |
| partition.timestamp-formatter | 将时间字符串转换为时间戳的格式串。 | 如果该参数没有设置，默认尝试yyyy-MM-dd HH:mm:ss与yyyy-MM-dd两个格式串。任何Java的DateTimeFormatter兼容的格式串都可以使用。 |

## 系统表

```sql
SELECT * FROM my_catalog.my_db.`my_table$snapshots`;
SELECT * FROM my_table$schemas;
SELECT * FROM my_table$options;
SELECT * FROM my_table$audit_log;
SELECT * FROM my_table$files;
SELECT * FROM my_table$files /*+ OPTIONS('scan.snapshot-id'='1') */;
SELECT * FROM my_table$tags;
SELECT * FROM my_table$branches;
SELECT * FROM my_table$consumers;
SELECT * FROM my_table$manifests /*+ OPTIONS('scan.snapshot-id'='1') */;
SELECT * FROM my_table$partitions;
SELECT * FROM my_table$aggregation_fields;
SELECT * FROM my_table$buckets;
SELECT * FROM my_table$ro; -- 高读取性能，但是可能是旧的数据，读取最新一次full compaction数据
```

## 标签管理

snapshot提供了一种方便我们查询历史数据，真实生产场景中Job 会产生很多snapshots,同时开发者会配置快照的过期策略。过期快照会被删除，过期快照的历史数据不能查询。

基于快照创建标签。比如每天创建一个标签、或者每小时创建一个标签 ,每天、每小时的历史数据以进行批量读取。

**标签适合于批读，一旦基于某个快照创建了Tag那么这个快照的元数据和数据文件就会被一直保留。**

| 参数                   | 说明                             | 备注                                                         |
| ---------------------- | -------------------------------- | ------------------------------------------------------------ |
| tag.automatic-creation | 是否自动创建标记。如何生成标签。 | none：没有自动创建的标签。 process-time：根据机器的时间,处理时间超过周期时间加上延迟，就创建TAG。 watermark：基于输入的水印，水印经过一段时间加上延迟，就创建TAG。 batch：在批处理场景中，任务完成后生成当前快照对应的标签。 |
| tag.creation-period    | 指定Tag创建时间间隔。            | daily:每天生成一个标签。 hourly:每小时生成一个标签。 two-hours:每两小时生成一个标签。 |
| tag.creation-delay     | 指定延迟时间。                   | 默认延迟时间为0。假如延迟为10分钟，到达需要创建Tag的时间点时，会再等待10分钟才创建Tag |
| tag.num-retained-max   | 指定最多保留的Tag数量。          | Tag会保留某个快照的元数据和数据文件，因旧的Tag可能导致不需要使用的历史数据文件占用存储。Tag数量超出这个参数时，最早的Tag将会被删除，数据也会被清理。 |



```sql
-- 手动创建
CALL sys.create_tag(`table` => 'database_name.table_name', tag => 'tag_name', [snapshot_id => <snapshot-id>]);
```



## Consumer ID

设置consumer id 后会在源表文件目录下创建consumer id  文件夹,保存对应consumer id将要消费到的snapshot

```SQL
{
  "nextSnapshot" : 29
}
```

piamon表判断快照（snapshot）是否过期清理时候，会去查看表对应文件系统下consumer文件夹下的所有消费者（Consumer Id），如果有消费者Id仍然依赖此快照，则快照不会Consumer Id没有消费完删除

当上一个作业停止时，新启动的作业可以继续使用上一个进度，而无需 从状态恢复。新读取的将从使用者文件中找到的下一个快照 ID 开始读取。 如果不希望此行为，则可以设置为 true。`'consumer.ignore-progress'`

```sql
insert into paimon.test.dwd_flink 
SELECT * FROM paimon.test.ods_flink /*+ OPTIONS('consumer-id' = 'myid1', 'consumer.expiration-time' = '1 d', 'consumer.mode' = 'exactly-once','consumer.ignore-progress' = 'true') */;

insert into paimon.test.dwd_flink 
SELECT * FROM paimon.test.ods_flink /*+ OPTIONS('consumer-id' = 'myid1', 'consumer.expiration-time' = '1 d', 'consumer.mode' = 'exactly-once') */;
```



## Branch & Fallback

### Branch



```sql
SELECT * FROM paimon.test.ods_flink /*+ OPTIONS('scan.snapshot-id' = '1') */;
CALL sys.create_tag('test.ods_flink', 'my_tag1', 1, '1 d'); -- 创建tag

SELECT * FROM paimon.test.ods_flink /*+ OPTIONS('scan.tag-name' = 'my_tag1') */;
CALL sys.create_branch('test.ods_flink', 'branch2', 'my_tag2'); -- 从tag创建分支
-- read from branch 'branch1'
SELECT * FROM paimon.test.`ods_flink$branch_branch2`;
update paimon.test.`ods_flink$branch_branch2` set age= 60;
select * from paimon.test.ods_flink
-- 将自定义分支快进到 main 将删除 main 分支中在分支的初始标记之后创建的所有快照、标签和架构。将快照、标签和架构从分支复制到主分支。
 CALL sys.fast_forward('test.ods_flink', 'branch2');
```

### Fallback

`scan.fallback-branch`：读取数据的时候批处理的从当前分支读取，如果分区中数据不存在则从fallback(退路)中读取。

对于流式读取作业，此功能目前不受支持

```sql
CALL sys.create_branch('test.ods_flink', 'streaming');  -- 创建branch

ALTER TABLE paimon.test.ods_flink SET ('scan.fallback-branch' = 'streaming'); -- 设置fallback 为streaming分支

ALTER TABLE paimon.test.`ods_flink$branch_streaming` SET ('bucket' = '1','changelog-producer' = 'input');

insert into paimon.test.`ods_flink$branch_streaming` select id,name,age,dt from default_catalog.default_database.input; -- 往stream分支流式写数据

select * from paimon.test.ods_flink; -- 查询不到会fallback到流式分支读取


insert overwrite paimon.test.ods_flink select id,name,age,dt from  paimon.test.`ods_flink$branch_streaming`;

update paimon.test.ods_flink set age = 70 ;

insert into  paimon.test.`ods_flink$branch_streaming`  values(100,'snapshots',20,'20241102');

ALTER TABLE paimon.test.ods_flink RESET ( 'scan.fallback-branch' );
```



## Dedicated Compaction

默认情况下 Paimon 写入器在写入记录时会根据需要进行 Compaction。

Paimon 支持并发写入不同的分区，如多个sql任务向同一张表同一个分区（同一个文件）写的时候，如果涉及到合并等提交就存在发生冲突的情况，导致作业异常，此时需要设置为 Dedicated Compaction。`write-only`设置为true，相当于关闭自动Compaction。



1. **压缩不会直接删除文件只有当快照关联的文件都已经被标记才会进行删除，所以压缩和快照过期当做整体来看。**
2. **压缩分为实时压缩、离线压缩（SET 'execution.runtime-mode' = 'batch'、-D executive.runtime mode=batch 控制）。**



### 实时压缩

```shell
=================库==============================
/opt/bigdata/flink19/bin/flink run \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --database test \
    --table ods_flink1
    
/opt/bigdata/flink19/bin/flink run \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --database test \
    --table ods_flink2   
    
=================表==============================
/opt/bigdata/flink19/bin/flink run \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact_database  \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --including_databases test 
    
    
=================================================
 -- compact table
CALL sys.compact(`table` => 'test.ods_flink1');
CALL sys.compact(`table` => 'test.ods_flink2');
-- compact table with options
CALL sys.compact(`table` => 'paimon.test.ods_flink', `options` => 'sink.parallelism=4');

```

### 离线压缩

```sql
SET 'execution.runtime-mode' = 'batch';
    /opt/bigdata/flink19/bin/flink run \
     -D execution.runtime-mode=batch \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --database test \ 
    --table ods_flink1 \
    --order_strategy zorder \
    --order_by name 

CALL sys.compact(`table` => 'test.ods_flink1', order_strategy => 'zorder', order_by => 'name')


  /opt/bigdata/flink19/bin/flink run \
     -D execution.runtime-mode=batch \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --database test \ 
    --table ods_flink1 \
    --partition_idle_time 1d
    =====================================================

    CALL sys.compact(`table` => 'test.ods_flink1', `partition_idle_time` => '1 s')
    ===========================================================
    /opt/bigdata/flink19/bin/flink run \
     -D execution.runtime-mode=batch \
    /opt/bigdata/flink19/lib/paimon-flink-action-0.9.0.jar \
    compact_database \
    --warehouse  hdfs://mj01:8020/lakehouse \
    --including_databases test \
    --partition_idle_time 1d 
    ================================================================
    CALL sys.compact_database('includingDatabases', 'mode', 'includingTables', 'excludingTables', 'tableOptions', 'partition_idle_time')
=============================================================================
CALL sys.compact_database('test', 'combined', '', '', '', '1 d')

```



## Table Mode

根据触发Compaction触发时机分为三种：

1. MOR （Merge On Read）：默认模式，仅执行次要压缩，读取需要合并。
2. COW （Copy On Write）：使用 ，将同步全量压缩，表示合并在写入时完成。`'full-compaction.delta-commits' = '1'`
3. MOW （Merge On Write）：在写入阶段使用 ，将查询 LSM 以生成 数据文件的 deletion vector 文件，读取时直接过滤掉不需要的行。`'deletion-vectors.enabled' = 'true'` 。写入过程中会引入额外的开销（查找 LSM Tree 并生成相应的 Deletion File）。 但在读取过程中，可以通过使用带有 deletion vectors 的数据来直接检索数据，从而避免了不同文件之间的额外合并成本







## Flink Insert Overwrite

实时只能insert into 

离线可以insert overwrite

`dynamic-partition-overwrite` :**默认为ture。true只会覆盖当前分区，false的时候所有数据清空保留select查询的数据**

```sql
insert overwrite paimon.test.ods_flink  select id,name,age,dt from paimon.test.ods_flink where dt = '20241024';

insert overwrite paimon.test.ods_flink /*+ OPTIONS('dynamic-partition-overwrite' = 'true') */ select id,name,age,dt from paimon.test.ods_flink where dt = '20241022';

```







## Paimon性能优化

1. 指定主键表中 指定`bucket-key`
2. 合理的bucket大小，200mb~1Gb
3. 异步compaction `num-sorted-run.stop-trigger = 2147483647`  `sort-spill-threshold = 10`  `lookup-wait = false`
4. Bloom-filter , bitmap ,  Deletion Vectors
5. `sink.parallelism` 尽量与slot一致
6. `file.compression.zstd-level` zstd算法压缩级别设置，默认1 ，最大支持9， 值越大，压缩效率越好，读取性能越差
7. 
