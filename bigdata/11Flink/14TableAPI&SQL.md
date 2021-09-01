# TableAPI&SQL

Apache Flink 有两种关系型 API 来做流批统一处理：Table API 和 SQL。Table API 是用于 Scala 和 Java 语言的查询API，它可以用一种非常直观的方式来组合使用选取、过滤、join 等关系型算子。Flink SQL 是基于 Apache Calcite 来实现的标准 SQL。

## 依赖

```xml
<!-- Either... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge_2.11</artifactId>
  <version>1.13.0</version>
  <scope>provided</scope>
</dependency>
<!-- or... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
  <version>1.13.0</version>
  <scope>provided</scope>
</dependency>


<!-- Either... (for the old planner that was available before Flink 1.9) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner_2.11</artifactId>
  <version>1.13.0</version>
  <scope>provided</scope>
</dependency>
<!-- or.. (for the new Blink planner) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner-blink_2.11</artifactId>
  <version>1.13.0</version>
  <scope>provided</scope>
</dependency>

<!--扩展依赖，可以自定义function或自定义解析-->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-common</artifactId>
  <version>1.13.2</version>
  <scope>provided</scope>
</dependency>
```

## 两种Planner区别

1. Blink 将批处理作业视作流处理的一种特例。严格来说，`Table` 和 `DataSet` 之间不支持相互转换，并且批处理作业也不会转换成 `DataSet` 程序而是转换成 `DataStream` 程序，流处理作业也一样。
2. Blink 计划器不支持 `BatchTableSource`，而是使用有界的 `StreamTableSource` 来替代。
3. 旧计划器和 Blink 计划器中 `FilterableTableSource` 的实现是不兼容的。旧计划器会将 `PlannerExpression` 下推至 `FilterableTableSource`，而 Blink 计划器则是将 `Expression` 下推。
4. 基于字符串的键值配置选项仅在 Blink 计划器中使用
5. `PlannerConfig` 在两种计划器中的实现（`CalciteConfig`）是不同的。
6. Blink 计划器会将多sink（multiple-sinks）优化成一张有向无环图（DAG），`TableEnvironment` 和 `StreamTableEnvironment` 都支持该特性。旧计划器总是将每个sink都优化成一个新的有向无环图，且所有图相互独立。
7. 旧计划器目前不支持 catalog 统计数据，而 Blink 支持。

### blink

不论输入数据源是流式的还是批式的，Table API 和 SQL 查询都会被转换成 DataStream程序。查询在内部表示为逻辑查询计划，并被翻译成两个阶段：

1. 优化逻辑执行计划
2. 翻译成 DataStream 程序

Table API 或者 SQL 查询在下列情况下会被翻译：

- 当 `TableEnvironment.executeSql()` 被调用时。该方法是用来执行一个 SQL 语句，一旦该方法被调用， SQL 语句立即被翻译。
- 当 `Table.executeInsert()` 被调用时。该方法是用来将一个表的内容插入到目标表中，一旦该方法被调用， TABLE API 程序立即被翻译。
- 当 `Table.execute()` 被调用时。该方法是用来将一个表的内容收集到本地，一旦该方法被调用， TABLE API 程序立即被翻译。
- 当 `StatementSet.execute()` 被调用时。`Table` （通过 `StatementSet.addInsert()` 输出给某个 `Sink`）和 INSERT 语句 （通过调用 `StatementSet.addInsertSql()`）会先被缓存到 `StatementSet` 中，`StatementSet.execute()` 方法被调用时，所有的 sink 会被优化成一张有向无环图。
- 当 `Table` 被转换成 `DataStream` 时。转换完成后，它就成为一个普通的 DataStream 程序，并会在调用 `StreamExecutionEnvironment.execute()` 时被执行。
- **从 1.11 版本开始，`sqlUpdate` 方法 和 `insertInto` 方法被废弃，从这两个方法构建的 Table 程序必须通过 `StreamTableEnvironment.execute()` 方法执行，而不能通过 `StreamExecutionEnvironment.execute()` 方法来执行。**

### old

Table API 和 SQL 查询会被翻译成 DataStream或者 DataSet程序， 这取决于它们的输入数据源是流式的还是批式的。查询在内部表示为逻辑查询计划，并被翻译成两个阶段：

1. 优化逻辑执行计划
2. 翻译成 DataStream 或 DataSet 程序

Table API 或者 SQL 查询在下列情况下会被翻译：

- 当 `TableEnvironment.executeSql()` 被调用时。该方法是用来执行一个 SQL 语句，一旦该方法被调用， SQL 语句立即被翻译。
- 当 `Table.executeInsert()` 被调用时。该方法是用来将一个表的内容插入到目标表中，一旦该方法被调用， TABLE API 程序立即被翻译。
- 当 `Table.execute()` 被调用时。该方法是用来将一个表的内容收集到本地，一旦该方法被调用， TABLE API 程序立即被翻译。
- 当 `StatementSet.execute()` 被调用时。`Table` （通过 `StatementSet.addInsert()` 输出给某个 `Sink`）和 INSERT 语句 （通过调用 `StatementSet.addInsertSql()`）会先被缓存到 `StatementSet` 中，`StatementSet.execute()` 方法被调用时，所有的 sink 会被优化成一张有向无环图。
- 对于 Streaming 而言，当`Table` 被转换成 `DataStream` 时触发翻译。转换完成后，它就成为一个普通的 DataStream 程序，并会在调用 `StreamExecutionEnvironment.execute()` 时被执行。对于 Batch 而言，`Table` 被转换成 `DataSet` 时触发翻译。转换完成后，它就成为一个普通的 DataSet 程序，并会在调用 `ExecutionEnvironment.execute()` 时被执行。
- **从 1.11 版本开始，`sqlUpdate` 方法 和 `insertInto` 方法被废弃。对于 Streaming 而言，如果一个 Table 程序是从这两个方法构建出来的，必须通过 `StreamTableEnvironment.execute()` 方法执行，而不能通过 `StreamExecutionEnvironment.execute()` 方法执行；对于 Batch 而言，如果一个 Table 程序是从这两个方法构建出来的，必须通过 `BatchTableEnvironment.execute()` 方法执行，而不能通过 `ExecutionEnvironment.execute()` 方法执行。**



## 创建TableEnvironment 

`TableEnvironment` 是 Table API 和 SQL 的核心概念。它负责:

- 在内部的 catalog 中注册 `Table`
- 注册外部的 catalog
- 加载可插拔模块
- 执行 SQL 查询
- 注册自定义函数 （scalar、table 或 aggregation）
- 将 `DataStream` 或 `DataSet` 转换成 `Table`
- 持有对 `ExecutionEnvironment` 或 `StreamExecutionEnvironment` 的引用

`Table` 总是与特定的 `TableEnvironment` 绑定。**不能在同一条查询中使用不同 TableEnvironment 中的表**，例如，对它们进行 join 或 union 操作。

```scala
// **********************
// FLINK STREAMING QUERY
// **********************
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.table.api.EnvironmentSettings
import org.apache.flink.table.api.bridge.scala.StreamTableEnvironment

val fsSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build()
val fsEnv = StreamExecutionEnvironment.getExecutionEnvironment
val fsTableEnv = StreamTableEnvironment.create(fsEnv, fsSettings)
// or val fsTableEnv = TableEnvironment.create(fsSettings)

// ******************
// FLINK BATCH QUERY
// ******************
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.table.api.bridge.scala.BatchTableEnvironment

val fbEnv = ExecutionEnvironment.getExecutionEnvironment
val fbTableEnv = BatchTableEnvironment.create(fbEnv)

// **********************
// BLINK STREAMING QUERY
// **********************
//1.13.0如不值当settings默认使用该方式创建
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.table.api.EnvironmentSettings
import org.apache.flink.table.api.bridge.scala.StreamTableEnvironment

val bsEnv = StreamExecutionEnvironment.getExecutionEnvironment
val bsSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build()
val bsTableEnv = StreamTableEnvironment.create(bsEnv, bsSettings)
// or val bsTableEnv = TableEnvironment.create(bsSettings)

// ******************
// BLINK BATCH QUERY
// ******************
import org.apache.flink.table.api.{EnvironmentSettings, TableEnvironment}

val bbSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build()
val bbTableEnv = TableEnvironment.create(bbSettings)

//如果只有一种planner的jar包，可以使用useAnyPlanner，  discovered automatically
```

## 创建表

`TableEnvironment` 维护着一个由标识符（identifier）创建的表 catalog 的映射。标识符由三个部分组成：catalog 名称、数据库名称以及对象名称。如果 catalog 或者数据库没有指明，就会使用当前默认值

`Table` 可以是虚拟的（视图 `VIEWS`）也可以是常规的（表 `TABLES`）。**视图 `VIEWS`可以从已经存在的`Table`中创建，一般是 Table API 或者 SQL 的查询结果。 表`TABLES`描述的是外部数据，例如文件、数据库表或者消息队列。**



**临时表（Temporary Table）和永久表（Permanent Table）**

表可以是临时的，并与单个 Flink 会话（session）的生命周期相关，也可以是永久的，并且在多个 Flink 会话和群集（cluster）中可见。

永久表需要 catalog（例如 Hive Metastore）以维护表的元数据。一旦永久表被创建，它将对任何连接到 catalog 的 Flink 会话可见且持续存在，直至被明确删除。

临时表通常**保存于内存中**并且仅在创建它们的 Flink 会话持续期间存在。这些表**对于其它会话是不可见的**。它们不与任何 catalog 或者数据库绑定但可以在一个命名空间（namespace）中创建。即使它们**对应的数据库被删除，临时表也不会被删除**。

使用与已存在的永久表相同的标识符去注册临时表。**临时表会屏蔽永久表**，并且只要临时表存在，永久表就无法访问。

```scala
// get a TableEnvironment
val tableEnv = ... // see "Create a TableEnvironment" section

// table is the result of a simple projection query 
val projTable: Table = tableEnv.from("X").select(...)

// register the Table projTable as table "projectedTable"
//创建临时视图
tableEnv.createTemporaryView("projectedTable", projTable)

//创建临时表
tableEnvironment
  .connect(...)
  .withFormat(...)
  .withSchema(...)
  .inAppendMode()
  .createTemporaryTable("MyTable")


//指定当前环境catelog,database
// get a TableEnvironment
val tEnv: TableEnvironment = ...;
tEnv.useCatalog("custom_catalog")
tEnv.useDatabase("custom_database")

val table: Table = ...;

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("exampleView", table)

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_database.exampleView", table)

// register the view named 'example.View' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("`example.View`", table)

// register the view named 'exampleView' in the catalog named 'other_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_catalog.other_database.exampleView", table)
```

从传统数据库系统的角度来看，`Table` 对象与 `VIEW` 视图非常像。也就是，定义了 `Table` 的查询是没有被优化的， 而且会被内嵌到另一个引用了这个注册了的 `Table`的查询中。如果多个查询都引用了同一个注册了的`Table`，那么它会被内嵌每个查询中并被执行多次， 也就是说注册了的`Table`的结果**不会**被共享（注：**Blink 计划器的`TableEnvironment`会优化成只执行一次**）。

## 查询表

Table API 是关于 Scala 和 Java 的集成语言式查询 API。与 SQL 相反，Table API 的查询不是由字符串指定，而是在宿主语言中逐步构建。

```scala
// get a TableEnvironment
val tableEnv = ... // see "Create a TableEnvironment" section

// register Orders table

// scan registered Orders table
val orders = tableEnv.from("Orders")
// compute revenue for all customers from France
//$号需要隐式引用
val revenue = orders
  .filter($"cCountry" === "FRANCE")
  .groupBy($"cID", $"cName")
  .select($"cID", $"cName", $"revenue".sum AS "revSum")

//使用sql
val revenue = tableEnv.sqlQuery("""
  |SELECT cID, cName, SUM(revenue) AS revSum
  |FROM Orders
  |WHERE cCountry = 'FRANCE'
  |GROUP BY cID, cName
  """.stripMargin)

```

## 输出表

`Table` 通过写入 `TableSink` 输出。`TableSink` 是一个通用接口，用于支持多种文件格式（如 CSV、Apache Parquet、Apache Avro）、存储系统（如 JDBC、Apache HBase、Apache Cassandra、Elasticsearch）或消息队列系统（如 Apache Kafka、RabbitMQ）。

批处理 `Table` 只能写入 `BatchTableSink`，而流处理 `Table` 需要指定写入 `AppendStreamTableSink`，`RetractStreamTableSink` 或者 `UpsertStreamTableSink`。

```scala
// get a TableEnvironment
val tableEnv = ... // see "Create a TableEnvironment" section

// create an output Table
val schema = new Schema()
    .field("a", DataTypes.INT())
    .field("b", DataTypes.STRING())
    .field("c", DataTypes.BIGINT())

tableEnv.connect(new FileSystem().path("/path/to/file"))
    .withFormat(new Csv().fieldDelimiter('|').deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("CsvSinkTable")

// compute a result Table using Table API operators and/or SQL queries
val result: Table = ...

// emit the result Table to the registered TableSink
result.executeInsert("CsvSinkTable")
```

## 解释表

Table API 提供了一种机制来解释计算 `Table` 的逻辑和优化查询计划。 这是通过 `Table.explain()` 方法或者 `StatementSet.explain()` 方法来完成的。

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tEnv = StreamTableEnvironment.create(env)

val table1 = env.fromElements((1, "hello")).toTable(tEnv, $"count", $"word")
val table2 = env.fromElements((1, "hello")).toTable(tEnv, $"count", $"word")
val table = table1
  .where($"word".like("F%"))
  .unionAll(table2)
println(table.explain())
```



## Demo

### DataStream与Table互转

```scala
import org.apache.flink.api.scala._
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.table.api.bridge.scala.StreamTableEnvironment

// create environments of both APIs
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = StreamTableEnvironment.create(env)

// create a DataStream
val dataStream = env.fromElements("Alice", "Bob", "John")

// interpret the insert-only DataStream as a Table
val inputTable = tableEnv.fromDataStream(dataStream)

// register the Table object as a view and query it
tableEnv.createTemporaryView("InputTable", inputTable)
val resultTable = tableEnv.sqlQuery("SELECT UPPER(f0) FROM InputTable")

// interpret the insert-only Table as a DataStream again
val resultStream = tableEnv.toDataStream(resultTable)

// add a printing sink and execute in DataStream API
resultStream.print()
env.execute()
```

### 