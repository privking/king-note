# HBASE API

## dependency

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>2.2.6</version>
</dependency>
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>2.2.6</version>
</dependency>
```

## 在resource加上hbase-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
/*
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
-->
<configuration>
    <!--
      The following properties are set for running HBase as a single process on a
      developer workstation. With this configuration, HBase is running in
      "stand-alone" mode and without a distributed file system. In this mode, and
      without further configuration, HBase and ZooKeeper data are stored on the
      local filesystem, in a path under the value configured for `hbase.tmp.dir`.
      This value is overridden from its default value of `/tmp` because many
      systems clean `/tmp` on a regular basis. Instead, it points to a path within
      this HBase installation directory.

      Running against the `LocalFileSystem`, as opposed to a distributed
      filesystem, runs the risk of data integrity issues and data loss. Normally
      HBase will refuse to run in such an environment. Setting
      `hbase.unsafe.stream.capability.enforce` to `false` overrides this behavior,
      permitting operation. This configuration is for the developer workstation
      only and __should not be used in production!__

      See also https://hbase.apache.org/book.html#standalone_dist
    -->
    <property>
        <name>hbase.tmp.dir</name>
        <value>/usr/local/soft/hbase/hbase-2.2.6/habase_temp_dir</value>
    </property>
    <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>

    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://node1:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
    <property>
        <name>hbase.master.port</name>
        <value>16000</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/usr/local/soft/zookeeper/apache-zookeeper-3.6.2-bin/data</value>
    </property>
</configuration>
```

## ConnectionUtil

```java
/**
 * @author king
 * TIME: 2020/11/3 - 23:14
 * 创建和关闭Connection对象
 * 可以使用HBaseConfiguration.create()，返回的Configuration,
 * 既包含hadoop8个配置文件的参数，又包含hbase-default.xml和hbase-site.xml中所有的参数配置！
 **/
public class ConnectionUtil {

    public static Connection getConn (){
        try {
            return ConnectionFactory.createConnection();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    public static void close(Connection conn) throws IOException {

        if (conn !=null) {
            conn.close();
        }

    }


}
```

## NameSpaceUtil

```java
public class NameSpaceUtil {
    private static Logger logger= LoggerFactory.getLogger(NameSpaceUtil.class);

    //查询所有的名称空间
    public static List<String> listNameSpace(Connection conn) throws IOException{
        List<String> nss=new ArrayList<>();
        //提供一个Admin
        Admin admin = conn.getAdmin();
        //查询所有的库
        NamespaceDescriptor[] namespaceDescriptors = admin.listNamespaceDescriptors();
        for (NamespaceDescriptor namespaceDescriptor : namespaceDescriptors) {
            //取出每个库描述中库的名称
            nss.add(namespaceDescriptor.getName());
        }
        //关闭admin
        admin.close();
        return nss;
    }

    //判断是否库存在
    public static boolean ifNSExists(Connection conn,String nsname) throws IOException {
        //库名校验
        if (StringUtils.isBlank(nsname)) {
            logger.error("请输入正常的库名！");
            //在后台提示，库名非法
            return false;
        }
        //提供一个Admin
        Admin admin = conn.getAdmin();
        //根据库名查询对应的NS，如果找不到就抛异常
        try {
            admin.getNamespaceDescriptor(nsname);
            return true;
        } catch (Exception e) {
            return false;
        }finally {
            //关闭admin
            admin.close();
        }
    }

    //创建库
    public static boolean creatNS(Connection conn,String nsname) throws IOException {
        //库名校验
        if (StringUtils.isBlank(nsname)) {
            logger.error("请输入正常的库名！");
            //在后台提示，库名非法
            return false;
        }
        //提供一个Admin
        Admin admin = conn.getAdmin();
        //新建库
        try {
            //先创建库的定义或描述
            NamespaceDescriptor descriptor = NamespaceDescriptor.create(nsname).build();
            admin.createNamespace(descriptor);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }finally {
            //关闭admin
            admin.close();
        }
    }

    //删除库
    public static boolean deleteNS(Connection conn,String nsname) throws IOException {
        //库名校验
        if (StringUtils.isBlank(nsname)) {
            logger.error("请输入正常的库名！");
            //在后台提示，库名非法
            return false;
        }

        //提供一个Admin
        Admin admin = conn.getAdmin();
        //只能删除空库，判断当前库是否为empty，不为空无法删除
        //查询当前库下有哪些表
        List<String> tables = getTablesInNameSpace(conn, nsname);
        if (tables.size()==0) {
            admin.deleteNamespace(nsname);
            //关闭admin
            admin.close();
            return true;
        }else {
            //关闭admin
            admin.close();
            logger.error(nsname+"库非空！无法删除！");
            return false;

        }
    }

    //查询库下有哪些表
    public static List<String> getTablesInNameSpace(Connection conn, String nsname) throws IOException {
        //库名校验
        if (StringUtils.isBlank(nsname)) {
            logger.error("请输入正常的库名！");
            //在后台提示，库名非法
            return null;
        }
        List<String> tables=new ArrayList<>();
        //提供一个Admin
        Admin admin = conn.getAdmin();
        //查询当前库所有的表
        List<TableDescriptor> tds = admin.listTableDescriptorsByNamespace(Bytes.toBytes(nsname));
        for (TableDescriptor td : tds) {
            tables.add(td.getTableName().getNameWithNamespaceInclAsString());
        }
        //关闭admin
        admin.close();
        return tables;
    }
}
```

## TableUtil

```java
public class TableUtil {
    private static Logger logger = LoggerFactory.getLogger(TableUtil.class);
    //验证表名是否合法并返回
    public static TableName checkTableName(String tableName, String nsname) {
        if (StringUtils.isBlank(tableName)) {
            logger.error("请输入正确的表名！");
            return null;
        }
        return TableName.valueOf(nsname, tableName);
    }
    //判断表是否存在
    public static boolean ifTableExists(Connection conn, String tableName, String nsname) throws IOException {
        //校验表名
        TableName tn = checkTableName(tableName, nsname);
        if (tn == null) {
            return false;
        }
        Admin admin = conn.getAdmin();
        //判断表是否存在,需要传入TableName对象
        boolean tableExists = admin.tableExists(tn);
        admin.close();
        return tableExists;
    }
    //创建表
    public static boolean createTable(Connection conn, String tableName, String nsname, String... cfs) throws IOException {
        //校验表名
        TableName tn = checkTableName(tableName, nsname);
        if (tn == null) {
            return false;
        }
        //至少需要传入一个列族
        if (cfs.length < 1) {
            logger.error("至少需要指定一个列族！");
            return false;
        }
        Admin admin = conn.getAdmin();
        //创建表的描述
        List<ColumnFamilyDescriptor> cfList = new ArrayList<>();
        for (String cf : cfs) {
            //TTL（Time-To-Live）：每个Cell的数据超时时间（当前时间 - 最后更新的时间）
            //MinVersion：如果当前存储的所有时间版本都早于TTL，至少MIN_VERSION个最新版本会保留下来。这样确保在你的查询以及数据早于TTL时有结果返回。
            ColumnFamilyDescriptor cfd = ColumnFamilyDescriptorBuilder
                    .newBuilder(Bytes.toBytes(cf))
                    //设置cell不过期
                    .setTimeToLive(HConstants.FOREVER)
                    .setMaxVersions(10)
                    .setMinVersions(3)
                    .build();
            cfList.add(cfd);
        }
        TableDescriptor td = TableDescriptorBuilder.newBuilder(tn)
                .setColumnFamilies(cfList)
                .build();

        //根据表的描述创建表
        admin.createTable(td);
        admin.close();
        return true;
    }
    //删除表
    public static boolean dropTable(Connection conn, String tableName, String nsname) throws IOException {
        //检查表是否存在
        if (!ifTableExists(conn, tableName, nsname)) {
            return false;
        }
        //校验表名
        TableName tn = checkTableName(tableName, nsname);
        Admin admin = conn.getAdmin();
        //删除之前需要先禁用表
        admin.disableTable(tn);
        //删除表
        admin.deleteTable(tn);
        admin.close();
        return true;
    }
}
```

## DataUtil

```java
/**
 * @author king
 * TIME: 2020/11/3 - 23:05
 *
 * 数据的增删改查，需要使用的是Table
 *
 * Put: 代表对单行数据的put操作
 * Get: 代表对单行数据的Get操作
 * Result: scan或get的单行的所有的记录
 * Cell： 代表一个单元格，hbase提供了CellUtil.clonexxx(Cell)，来获取cell中的列族、列名和值属性
 *
 * 在hbase中，操作的数据都是以byte[]形式存在，需要把常用的数据类型转为byte[]
 * hbase提供了Bytes工具类
 * Bytes.toBytes(x): 基本数据类型转byte[]
 * Bytes.toXxx(x): 从byte[]转为Xxx类型！
 **/
public class DataUtil {
    //先获取到表的table对象
    public static Table getTable(Connection conn, String tableName, String nsname) throws IOException {
        //验证表名是否合法
        TableName tn = TableUtil.checkTableName(tableName, nsname);
        if (tn == null) {
            return null;
        }
        //根据TableName获取对应的Table
        return conn.getTable(tn);
    }
    //put 表名,rowkey,列名(列族名:列名),value
    public static void put(Connection conn,String tableName,String nsname,String rowkey,String cloumnFamily,
                           String cloumnQualifier,String value) throws IOException {
        //获取表对象
        Table table = getTable(conn, tableName, nsname);
        if (table==null) {
            return;
        }
        //创建一个Put对象
        Put put = new Put(Bytes.toBytes(rowkey));
        //向put中设置cell的细节信息
        put.addColumn(Bytes.toBytes(cloumnFamily), Bytes.toBytes(cloumnQualifier), Bytes.toBytes(value));
        //.addColumn(family, qualifier, value)
        table.put(put);
        table.close();
    }
    // get 表名   rowkey
    public static void get(Connection conn,String tableName,String nsname,String rowkey) throws IOException {
        //获取表对象
        Table table = getTable(conn, tableName, nsname);
        if (table==null) {
            return ;
        }
        Get get = new Get(Bytes.toBytes(rowkey));
        //设置单行查询的详细信息
        //设置查哪个列
        //get.addColumn(family, qualifier)
        //设置查哪个列族
        //get.addFamily(family)
        //只查某个时间戳的数据
        //get.setTimeStamp(timestamp)
        //设置返回的versions
        //get.setMaxVersions(maxVersions)
        Result result = table.get(get);
        //System.out.println(result);
        parseResult(result);
        table.close();
    }
    //遍历result
    public static void parseResult(Result result) {
        if (result != null) {
            Cell[] cells = result.rawCells();
            for (Cell cell : cells) {
                System.out.println("行："+ Bytes.toString(CellUtil.cloneRow(cell))+
                        "  列族："+Bytes.toString(CellUtil.cloneFamily(cell))+"   列名："+
                        Bytes.toString(CellUtil.cloneQualifier(cell))+
                        "  值:"+Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
    }
    
    
     public static void scan(Connection conn, String tableName, String nsname) throws Exception {
        Table table = getTable(conn, tableName, nsname);

        if (table == null) {
            return;
        }

        Scan scan = new Scan();
        //起始行
        scan.withStartRow(Bytes.toBytes(1));
        //结束行
        scan.withStopRow(Bytes.toBytes(50));

//        scan.addColumn()
//        scan.addFamily()
         //设置过滤器
//        scan.setFilter()
//        scan.setLimit()
//        scan.setxxxxxx
        //多行result的集合
        ResultScanner scanner = table.getScanner(scan);

        Iterator<Result> iterator = scanner.iterator();
        while (iterator.hasNext()) {
            Result next = iterator.next();
            parseResult(next);

        }
        table.close();
    }


    public static void delete(Connection conn, String tableName, String nsname, String rowKey) throws Exception {
        Table table = getTable(conn, tableName, nsname);

        if (table == null) {
            return;
        }

        Delete delete = new Delete(Bytes.toBytes(rowKey));
//        删除最新 version
//        delete.addColumn()
//        删除all version
//        delete.addColumns()
//        delete.addFamily()
        table.delete(delete);
        table.close();
    }
}
```