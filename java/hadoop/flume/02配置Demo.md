# 配置Demo

## netcatsource-loggingsink

* netcat source:  作用就是监听某个tcp端口手动的数据，将每行数据封装为一个event。
  工作原理类似于`nc -l  -k 端口`

![image-20201022234110663](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201022234110663-1603381270-7bbcde.png)

* logger sink: 作用使用logger(日志输出器)将event输出到文件或控制台,使用info级别记录event

![image-20201022234232102](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201022234232102-1603381352-f96422.png)



* memery channel

![image-20201022234339160](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201022234339160-1603381419-ded7f2.png)

```properties
# sh flume-ng agent -n a1 -f ../conf/netcatsource-loggersink.conf -c ../conf 
# netcatsource-loggingsink.conf
# a1是agent的名称，a1中定义了一个叫r1的source，如果有多个，使用空格间隔
a1.sources = r1
a1.sinks = k1
a1.channels = c1
#组名名.属性名=属性值
a1.sources.r1.type=netcat
a1.sources.r1.bind=hadoop102
a1.sources.r1.port=44444

#定义sink
a1.sinks.k1.type=logger
a1.sinks.k1.maxBytesToLog=100

#定义chanel
a1.channels.c1.type=memory
a1.channels.c1.capacity=1000

#连接组件 同一个source可以对接多个channel，一个sink只能从一个channel拿数据！
a1.sources.r1.channels=c1
a1.sinks.k1.channel=c1
```

![image-20201022234442399](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201022234442399-1603381482-677c51.png)

![image-20201022234507340](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201022234507340-1603381507-b41179.png)

## execsource-hdfssink

- EXECSource

  - execsource会在agent启动时，运行一个linux命令，运行linux命令的进程要求是一个可以持续产生数据的进程！
    将标准输出的数据封装为event!
  - 通常情况下，如果指定的命令退出了，那么source也会退出并且不会再封装任何的数据！
  - 所以使用这个source一般推荐类似cat ,tail -f 这种命令

  ![image-20201023001822895](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201023001822895-1603383503-bce493.png)

-  HDFSSink

  - https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#hdfs-sink
  - hdfssink将event写入到HDFS！目前只支持生成两种类型的文件： text | sequenceFile,这两种文件都可以使用压缩！
  - 写入到HDFS的文件可以自动滚动（关闭当前正在写的文件，创建一个新文件）。基于时间、events的数量、数据大小进行周期性的滚动！ 
  -  支持基于时间和采集数据的机器进行分桶和分区操作！
  -  HDFS数据所上传的目录或文件名可以包含一个格式化的转义序列，这个路径或文件名会在上传event时，被自动替换，替换为完整的路径名！
  - 使用此Sink要求本机已经安装了hadoop，或持有hadoop的jar包！

  ![image-20201023002209724](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201023002209724-1603383729-3a3e6c.png)

  | Name                   | Default      | Description                                                  |
  | :--------------------- | :----------- | :----------------------------------------------------------- |
  | **channel**            | –            |                                                              |
  | **type**               | –            | The component type name, needs to be `hdfs`                  |
  | **hdfs.path**          | –            | HDFS directory path (eg hdfs://namenode/flume/webdata/)      |
  | hdfs.filePrefix        | FlumeData    | Name prefixed to files created by Flume in hdfs directory    |
  | hdfs.fileSuffix        | –            | Suffix to append to file (eg `.avro` - *NOTE: period is not automatically added*) |
  | hdfs.inUsePrefix       | –            | Prefix that is used for temporal files that flume actively writes into |
  | hdfs.inUseSuffix       | `.tmp`       | Suffix that is used for temporal files that flume actively writes into |
  | hdfs.emptyInUseSuffix  | false        | If `false` an `hdfs.inUseSuffix` is used while writing the output. After closing the output `hdfs.inUseSuffix` is removed from the output file name. If `true` the `hdfs.inUseSuffix` parameter is ignored an empty string is used instead. |
  | hdfs.rollInterval      | 30           | Number of seconds to wait before rolling current file (0 = never roll based on time interval) |
  | hdfs.rollSize          | 1024         | File size to trigger roll, in bytes (0: never roll based on file size) |
  | hdfs.rollCount         | 10           | Number of events written to file before it rolled (0 = never roll based on number of events) |
  | hdfs.idleTimeout       | 0            | Timeout after which inactive files get closed (0 = disable automatic closing of idle files) |
  | hdfs.batchSize         | 100          | number of events written to file before it is flushed to HDFS |
  | hdfs.codeC             | –            | Compression codec. one of following : gzip, bzip2, lzo, lzop, snappy |
  | hdfs.fileType          | SequenceFile | File format: currently `SequenceFile`, `DataStream` or `CompressedStream` (1)DataStream will not compress output file and please don’t set codeC (2)CompressedStream requires set hdfs.codeC with an available codeC |
  | hdfs.maxOpenFiles      | 5000         | Allow only this number of open files. If this number is exceeded, the oldest file is closed. |
  | hdfs.minBlockReplicas  | –            | Specify minimum number of replicas per HDFS block. If not specified, it comes from the default Hadoop config in the classpath. |
  | hdfs.writeFormat       | Writable     | Format for sequence file records. One of `Text` or `Writable`. Set to `Text` before creating data files with Flume, otherwise those files cannot be read by either Apache Impala (incubating) or Apache Hive. |
  | hdfs.threadsPoolSize   | 10           | Number of threads per HDFS sink for HDFS IO ops (open, write, etc.) |
  | hdfs.rollTimerPoolSize | 1            | Number of threads per HDFS sink for scheduling timed file rolling |
  | hdfs.kerberosPrincipal | –            | Kerberos user principal for accessing secure HDFS            |
  | hdfs.kerberosKeytab    | –            | Kerberos keytab for accessing secure HDFS                    |
  | hdfs.proxyUser         |              |                                                              |
  | hdfs.round             | false        | Should the timestamp be rounded down (if true, affects all time based escape sequences except %t) |
  | hdfs.roundValue        | 1            | Rounded down to the highest multiple of this (in the unit configured using `hdfs.roundUnit`), less than current time. |
  | hdfs.roundUnit         | second       | The unit of the round down value - `second`, `minute` or `hour`. |
  | hdfs.timeZone          | Local Time   | Name of the timezone that should be used for resolving the directory path, e.g. America/Los_Angeles. |
  | hdfs.useLocalTimeStamp | false        | Use the local time (instead of the timestamp from the event header) while replacing the escape sequences. |
  | hdfs.closeTries        | 0            | Number of times the sink must try renaming a file, after initiating a close attempt. If set to 1, this sink will not re-try a failed rename (due to, for example, NameNode or DataNode failure), and may leave the file in an open state with a .tmp extension. If set to 0, the sink will try to rename the file until the file is eventually renamed (there is no limit on the number of times it would try). The file may still remain open if the close call fails but the data will be intact and in this case, the file will be closed only after a Flume restart. |
  | hdfs.retryInterval     | 180          | Time in seconds between consecutive attempts to close a file. Each close call costs multiple RPC round-trips to the Namenode, so setting this too low can cause a lot of load on the name node. If set to 0 or less, the sink will not attempt to close the file if the first attempt fails, and may leave the file open or with a ”.tmp” extension. |
  | serializer             | `TEXT`       | Other possible options include `avro_event` or the fully-qualified class name of an implementation of the `EventSerializer.Builder` interface. |
  | serializer.*           |              |                                                              |

  ```properties
  #a1是agent的名称，a1中定义了一个叫r1的source，如果有多个，使用空格间隔
  a1.sources = r1
  a1.sinks = k1
  a1.channels = c1
  #组名名.属性名=属性值
  a1.sources.r1.type=exec
  a1.sources.r1.command=tail -f /opt/module/hive/logs/hive.log
  
  #定义chanel
  a1.channels.c1.type=memory
  a1.channels.c1.capacity=1000
  
  #定义sink
  a1.sinks.k1.type = hdfs
  #一旦路径中含有基于时间的转义序列，要求event的header中必须有timestamp=时间戳，如果没有需要将useLocalTimeStamp = true
  a1.sinks.k1.hdfs.path = hdfs://node1:9000/flume/%Y%m%d/%H/%M
  #上传文件的前缀
  a1.sinks.k1.hdfs.filePrefix = logs-
  
  #以下三个和目录的滚动相关，目录一旦设置了时间转义序列，基于时间戳滚动
  #是否将时间戳向下舍
  a1.sinks.k1.hdfs.round = true
  #多少时间单位创建一个新的文件夹
  a1.sinks.k1.hdfs.roundValue = 10
  #重新定义时间单位
  a1.sinks.k1.hdfs.roundUnit = minute
  
  #是否使用本地时间戳
  a1.sinks.k1.hdfs.useLocalTimeStamp = true
  #积攒多少个Event才flush到HDFS一次
  a1.sinks.k1.hdfs.batchSize = 100
  
  #以下三个和文件的滚动相关，以下三个参数是或的关系！以下三个参数如果值为0都代表禁用！
  #60秒滚动生成一个新的文件
  a1.sinks.k1.hdfs.rollInterval = 60
  #设置每个文件到128M时滚动
  a1.sinks.k1.hdfs.rollSize = 134217700
  #每写多少个event滚动一次
  a1.sinks.k1.hdfs.rollCount = 0
  
  
  #连接组件 同一个source可以对接多个channel，一个sink只能从一个channel拿数据！
  a1.sources.r1.channels=c1
  a1.sinks.k1.channel=c1
  ```


  