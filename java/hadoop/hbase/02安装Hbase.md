# 安装Hbase

1.下载hbase 2.2.6 

2.解压

3.配置

hbase-site.xml

```xml
<configuration>
    <!--hbase-site.xml-->
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

regionservers(配置集群节点)

```
node1
node2
node3
```

hbase-env.sh

```sh
# 不使用自带zk
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/usr/local/jdk1.8.0_161
```

