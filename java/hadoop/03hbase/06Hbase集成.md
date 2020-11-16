# Hbase集成

## 集成MR

统计的需要:HBase的数据都是分布式存储在RegionServer上的，所以对于类似传统关系型数据库的group by操作，扫描器是无能为力的，只有当所有结果都返回到客户端的时候，才能进行统计。这样做一是慢，二是会产生很大的网络开销，所以使用MapReduce在服务器端就进行统计是比较好的方案。

性能的需要：说白了就是“快”！如果遇到较复杂的场景，在扫描器上添加多个过滤器后，扫描的性能很低；或者当数据量很大的时候扫描器也会执行得很慢，原因是扫描器和过滤器内部实现的机制很复杂，虽然使用者调用简单，但是服务器端的性能就不敢保证了

### 加入依赖

```xml
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <encoding>UTF-8</encoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>


    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs-client -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs-client</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-client -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.3.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-mapreduce-client-core -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-core -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.2.1</version>
            <scope>provided</scope>
        </dependency>

        <!--<dependency>-->
        <!--<groupId>org.apache.hbase</groupId>-->
        <!--<artifactId>hbase-server</artifactId>-->
        <!--<version>2.2.6</version>-->
        <!--</dependency>-->
        <!--<dependency>-->
        <!--<groupId>org.apache.hbase</groupId>-->
        <!--<artifactId>hbase-client</artifactId>-->
        <!--<version>2.2.6</version>-->
        <!--</dependency>-->

        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase</artifactId>
            <version>2.2.6</version>
            <type>pom</type>
        </dependency>


        <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-shaded-mapreduce -->
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-shaded-mapreduce</artifactId>
            <version>2.2.6</version>
        </dependency>


    </dependencies>
```

### Mapper

```java
/**
 * @author king
 * TIME: 2020/11/7 - 12:45
 *
 * TableMapper 简单继承了Mapper 主要是hbase输入类型为ImmutableBytesWritable, Result
 * ImmutableBytesWritable -> cloumnkey
 **/
public class HbaseMapper extends TableMapper<ImmutableBytesWritable, Put> {

    @Override
    protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
        Put put = new Put(key.get());
        //遍历添加column行
        for(Cell cell: value.rawCells()){
            put.addColumn(CellUtil.cloneFamily(cell), CellUtil.cloneQualifier(cell), CellUtil.cloneValue(cell));
        }
        //将读取到的每行数据写入到context中作为map的输出
        context.write(key, put);
    }
}

```

### Reducer

```java
/**
 * @author king
 * TIME: 2020/11/7 - 12:58
 **/
public class HbaseReducer extends TableReducer<ImmutableBytesWritable, Put, ImmutableBytesWritable> {

    Logger log = LoggerFactory.getLogger(HbaseReducer.class);
    @Override
    protected void reduce(ImmutableBytesWritable key, Iterable<Put> values, Context context) throws IOException, InterruptedException {
        //读出来的每一行数据写入到fruit_mr表中
        for(Put put: values){
            log.info(Bytes.toString(key.get()));
            List<Cell> cells = put.get(Bytes.toBytes("info"), Bytes.toBytes("name"));
            cells.forEach(cell->{
                log.info(Bytes.toString(CellUtil.cloneFamily(cell)));
                log.info(Bytes.toString(CellUtil.cloneQualifier(cell)));
                log.info(Bytes.toString(CellUtil.cloneValue(cell)));
            });
            context.write(key, put);
        }
    }
}
```

### Driver

```java
public class HbaseDriver extends Configured implements Tool {

    public int run(String[] args) throws Exception {
        //得到Configuration
        Configuration conf = this.getConf();
        //创建Job任务
        Job job = Job.getInstance(conf, this.getClass().getSimpleName());
        job.setJarByClass(HbaseDriver.class);

        //配置Job
        Scan scan = new Scan();
        scan.setCacheBlocks(false);
        scan.setCaching(500);

        //设置Mapper，注意导入的是mapreduce包下的，不是mapred包下的，后者是老版本
        TableMapReduceUtil.initTableMapperJob(
                "java_create:mr_test1", //数据源的表名
                scan, //scan扫描控制器
                HbaseMapper.class,//设置Mapper类
                ImmutableBytesWritable.class,//设置Mapper输出key类型
                Put.class,//设置Mapper输出value值类型
                job//设置给哪个JOB
        );
        //设置Reducer
        TableMapReduceUtil.initTableReducerJob("java_create:mr_test2", HbaseReducer.class, job);
        //设置Reduce数量，最少1个
        job.setNumReduceTasks(1);

        boolean isSuccess = job.waitForCompletion(true);
        if (!isSuccess) {
            throw new IOException("Job running with error");
        }
        return isSuccess ? 0 : 1;
    }

    public static void main(String[] args) throws Exception {
        // Let <code>ToolRunner</code> handle generic command-line options
        int res = ToolRunner.run(new Configuration(), new HbaseDriver(), args);
        System.exit(res);
    }

}
```

### 执行

将hbase-site.xml 拷贝到hadoop中

打包后运行：

```sh
HADOOP_CLASSPATH=`/usr/local/soft/hbase/hbase-2.2.6/bin/hbase mapredcp` hadoop jar hbase_mr_test-1.0-SNAPSHOT.jar priv.king.HbaseDriver -libjars $(/usr/local/soft/hbase/hbase-2.2.6/bin/hbase mapredcp | tr ':' ',') ...
```

### 注意

- 当mapper阶段value为 Put类型输出key相同时，有combiner过程，合并
- ImmutableBytesWritable，Put等都没有实现 WritableComparable ,也能序列化，是因为在`TableMapReduceUtil.initTableReducerJob("java_create:mr_test2", HbaseReducer.class, job);`时手动设置了自定义序列化器
- `conf.setStrings("io.serializations", new String[]{conf.get("io.serializations"), MutationSerialization.class.getName(), ResultSerialization.class.getName()});`

## 集成Hive

通过 Hive 与 HBase 整合，可以将 HBase 的数据通过 Hive 来分析，让 HBase 支持 JOIN、GROUP 等 SQL 查询语法。

实现将批量数据导入到 HBase 表中。

### 依赖

已有 HDFS、MapReduce、Hive、Zookeeper、HBase 环境。

确保 Hive 的 lib 目录下有 hive-hbase-handler-xxx.jar、Zookeeper jar、HBase Server jar、HBase Client jar 包

将hbase-site.xml 放到hive下

### Hbase表存在

```sql
create external table hbase_t1(
    id string,
    name string,
    age string,
    sex string,
    friends string
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:name,info:age,info:sex,info:friends")
TBLPROPERTIES ("hbase.table.name" = "java_create:mr_test2");

--TBLPROPERTIES ("hbase.table.name" = "java_create:mr_test2"); 可选 默认在default namespace下创建同名表
-- :key ->对应cloumnKey
-- 指定为外部表，因为hbase中存在了
```

### Hbase表不存在

```sql
create table hbase_t1(
    id string,
    name string,
    age string,
    sex string,
    friends string
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:name,info:age,info:sex,info:friends")
TBLPROPERTIES ("hbase.table.name" = "java_create:mr_test2");

--指定为管理表
```

- managed_table
  - 由hive的表管理数据的生命周期！在hive中，执行droptable时，
  - 不仅将hive中表的元数据删除，还把表目录中的数据删除
- external_table
  - hive表不负责数据的生命周期！在hive中，执行droptable时，
  - 只会将hive中表的元数据删除，不把表目录中的数据删除
- Storage Handlers
  - Storage Handlers是一个扩展模块，帮助hive分析不在hdfs存储的数据！
  - 例如数据存储在hbase上，可以使用hive提供的对hbase的Storage Handlers，来读写hbase中的数据！
- native table：
  - 本地表！ hive无需通过Storage Handlers就能访问的表。
  - [ROW FORMAT row_format] [STORED AS file_format]   file_format: ORC|TEXTFILE|SEQUNCEFILE|PARQUET
- non-native table : 
  - hive必须通过Storage Handlers才能访问的表！
  - STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]
- SERDE：hive中序列化器和反序列化器
  - 表中的数据是什么样的格式，就必须使用什么样的SerDe!
  - 纯文本：  row format delimited ，默认使用LazySimpleSerDe
  - JSON格式：  使用JsonSerde
  - ORC：    使用读取ORC的SerDe
  - Paquet:  使用读取PaquetSerDe
  - HDFS files –> InputFileFormat –> <key, value> –> Deserializer –> Row object
    Row object –> Serializer –> <key, value> –> OutputFileFormat –> HDFS files
  - 调用InputFormat，将文件切成不同的文档。每篇文档即一行(Row)。
    调用SerDe的Deserializer，将一行(Row)，切分为各个字段。

```sql
create table testSerde2(
name string,
friends array<string>
)
ROW FORMAT SERDE 
'org.apache.hive.hcatalog.data.JsonSerDe' 
STORED AS TEXTFILE
```

