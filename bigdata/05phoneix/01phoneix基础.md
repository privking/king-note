# phoneix基础

- 为低延迟的应用程序提供了OLTP和Hadoop操作分析，Apache Phoenix通过结合两者的优点：
  - 标准SQL和JDBC api的强大功能和完整的ACID事务处理能力
  - 通过利用HBase作为它的存储支持，NoSQL提供的 late-bound, schema-on-read的灵活性
- 能够让我们使用标准的 JDBC API 去建表, 插入数据和查询 HBase 中的数据, 从而可以避免使用 HBase 的客户端 API.

## 特点

1. 将 SQl 查询编译为 HBase 扫描

2. 确定扫描 Rowkey 的最佳开始和结束位置

3. 扫描并行执行

4. 将 where 子句推送到服务器端的过滤器

5. 通过协处理器进行聚合操作（就是在流程前后执行操作）

6. 完美支持 HBase 二级索引创建

7. DML命令以及通过DDL命令创建和操作表和版本化增量更改。

8. 容易集成：如Spark，Hive，Pig，Flume和Map Reduce。



![image-20201107234515830](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201107234515830-1604763923-9e0850.png)



## Phoneix数据存储

Phoenix 将 HBase 的数据模型映射到关系型世界

![image-20201107234614556](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201107234614556-1604763974-792114.png)

## 安装：

- 下载解压
- 将解压后的client和server拷贝到hbase/lib中
- 配置环境变量
- 配置python2.7环境
- `sqlline.py hadoop201,hadoop202,hadoop203:2181`

```sh
export PHOENIX_HOME=/xxxxxxxxxxxx
export PHOENIX_CLASSPATH=$PHOENIX_HOME
export PATH=$PATH:$PHOENIX_HOME/bin
```

