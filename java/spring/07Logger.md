# Logger
## 常用日志组件
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203823-59d149.png)
## 日志框架
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203839-72b0f4.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203864-4505ba.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203887-3f2a7e.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203904-e45d5f.png)

## slf4j适配器
* 适配器需要去除原来的jar,适配器中包含原来的经过修改的jar
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203923-5c77c4.png)

## spring 4.x日志
* 直接依赖commons-logging
* 如果有配置文件配置Log实现等等，优先使用
* 如果没有按照顺序查找
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203942-22f671.png)

* 所以要使用log4j直接添加依赖，即可

```xml
 <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```


## spring 5.x日志
* 新模块spring-jcl，对commons-logging修改,所以直接加log4j不行
* 支持log4j2，slf4j,jul
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203962-a230f2.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203976-ae9b7e.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203996-b42856.png)


## MyBatis日志
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204031-02fb1b.png)
* 加载顺序如上图
* 与spring整合后如果不是用的slf4j，则会进入jcl(spring),如果没有使用其他日志框架，就会使用jul,jul默认级别为info,就不会打印sql.
* 整合spring5.x后直接加入log4j的jar也不能打印
* 可以调用usexxxx方法手动设置
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204072-5c8397.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204085-64584d.png)


## 配置文件
* log4j.properties

```
### set log levels 日志的优先级###
log4j.rootLogger=debug , console , fileDebug , fileError

### console ###
##控制台显示
log4j.appender.console = org.apache.log4j.ConsoleAppender 
log4j.appender.console.Target = System.out
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern =  %-d{yyyy-MM-dd HH\:mm\:ss} [%p]-[%c] %m%n

### log file 保存日志文件###
## 每天产生一个文件
log4j.appender.fileDebug = org.apache.log4j.DailyRollingFileAppender 
log4j.appender.fileDebug.File = D:/logs/spring_debug.log 
## 信息是增加，不是覆盖
log4j.appender.fileDebug.Append = true 
log4j.appender.fileDebug.Threshold = DEBUG
log4j.appender.fileDebug.layout = org.apache.log4j.PatternLayout
log4j.appender.fileDebug.layout.ConversionPattern = %-d{yyyy-MM-dd HH\:mm\:ss} [%p]-[%c] %m%n

### exception 保存异常文件###
log4j.appender.fileError = org.apache.log4j.DailyRollingFileAppender
log4j.appender.fileError.File = D:/logs/spring_error.log 
log4j.appender.fileError.Append = true
log4j.appender.fileError.Threshold = ERROR
log4j.appender.fileError.layout = org.apache.log4j.PatternLayout
log4j.appender.fileError.layout.ConversionPattern = %-d{yyyy-MM-dd HH\:mm\:ss} [%p]-[%c] %m%n

#日志文件最大值
log4j.appender.File.MaxFileSize = 1000MB

```
* logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration  scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property name="log.path" value="D:/logs/logback" />
    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
             <level>ERROR</level>
         </filter>-->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!--输出到文件-->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/logback.%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="console" />
        <appender-ref ref="file" />
    </root>
    
</configuration>
```