# Azkaban简单使用

## 安装

- azkaban-web-server-2.5.0.tar.gz  服务器
- azkaban-executor-server-2.5.0.tar.gz 执行服务器
- azkaban-sql-script-2.5.0.tar.gz sql脚本

## mysql创建表

在解压安装后，使用azkaban提供的sql脚本创建表

```ruby
mysql -uroot -p
mysql> create database azkaban;
mysql> use azkaban;
Database changed
mysql> source /home/azkaban/azkaban-2.5.0/create-all-sql-2.5.0.sql;
mysql> show tables;
+------------------------+
| Tables_in_azkaban      |
+------------------------+
| active_executing_flows |
| active_sla             |
| execution_flows        |
| execution_jobs         |
| execution_logs         |
| project_events         |
| project_files          |
| project_flows          |
| project_permissions    |
| project_properties     |
| project_versions       |
| projects               |
| properties             |
| schedules              |
| triggers               |
+------------------------+
15 rows in set (0.00 sec)
```

## 配置时区

这个配置需要给集群的每个主机设置，因为任务调度离不开准确的时间。也可以直接把相关文件拷贝到别的主机作覆盖。

```csharp
[root@s166 azkaban]# tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.
#? 5
Please select a country.
 1) Afghanistan       18) Israel            35) Palestine
 2) Armenia       19) Japan         36) Philippines
 3) Azerbaijan        20) Jordan            37) Qatar
 4) Bahrain       21) Kazakhstan        38) Russia
 5) Bangladesh        22) Korea (North)     39) Saudi Arabia
 6) Bhutan        23) Korea (South)     40) Singapore
 7) Brunei        24) Kuwait            41) Sri Lanka
 8) Cambodia          25) Kyrgyzstan        42) Syria
 9) China         26) Laos          43) Taiwan
10) Cyprus        27) Lebanon           44) Tajikistan
11) East Timor        28) Macau         45) Thailand
12) Georgia       29) Malaysia          46) Turkmenistan
13) Hong Kong         30) Mongolia          47) United Arab Emirates
14) India         31) Myanmar (Burma)       48) Uzbekistan
15) Indonesia         32) Nepal         49) Vietnam
16) Iran          33) Oman          50) Yemen
17) Iraq          34) Pakistan
#? 9
Please select one of the following time zone regions.
1) Beijing Time
2) Xinjiang Time
#? 1

The following information has been given:

    China
    Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Local time is now:  Sat Jul 28 18:29:58 CST 2018.
Universal Time is now:  Sat Jul 28 10:29:58 UTC 2018.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
    TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai
```

## 修改配置


/webserver/conf目录下的azkaban.properties

```properties
#Azkaban Personalization Settings
azkaban.name=Test
azkaban.label=My Local Azkaban
azkaban.color=#FF3601
azkaban.default.servlet.path=/index
web.resource.dir=web/
default.timezone.id=Asia/Shanghai

#Azkaban UserManager class
user.manager.class=azkaban.user.XmlUserManager
user.manager.xml.file=conf/azkaban-users.xml

#Loader for projects
executor.global.properties=conf/global.properties
azkaban.project.dir=projects

database.type=mysql
mysql.port=3306
mysql.host=localhost
mysql.database=azkaban
mysql.user=root
mysql.password=root
mysql.numconnections=100

# Velocity dev mode
velocity.dev.mode=false

# Azkaban Jetty server properties.
jetty.maxThreads=25
jetty.ssl.port=8443
jetty.port=8081
jetty.keystore=keystore
jetty.password=password
jetty.keypassword=jiaoroot
jetty.truststore=keystore
jetty.trustpassword=password

# Azkaban Executor settings
executor.port=12321

# mail settings
mail.sender=xxxxxxx@qq.com
mail.host=smtp.qq.com
job.failure.email=
job.success.email=

lockdown.create.projects=false

cache.directory=cache
```

修改/conf/目录下的azkaban-users.xml

```xml
<azkaban-users>
        <user username="azkaban" password="azkaban" roles="admin" groups="azkaban" />
        <user username="metrics" password="metrics" roles="metrics"/>
        <user username="admin" password="admin" roles="admin">
        
        <role name="admin" permissions="ADMIN" />
        <role name="metrics" permissions="METRICS"/>
</azkaban-users>
```

修改/executor/conf目录下的azkaban.properties

```properties
#Azkaban
default.timezone.id=Asia/Shanghai

# Azkaban JobTypes Plugins
azkaban.jobtype.plugin.dir=plugins/jobtypes

#Loader for projects
executor.global.properties=conf/global.properties
azkaban.project.dir=projects

database.type=mysql
mysql.port=3306
mysql.host=localhost
mysql.database=azkaban
mysql.user=root
mysql.password=root
mysql.numconnections=100

# Azkaban Executor settings
executor.maxThreads=50
executor.port=12321
executor.flow.threads=30
```

## 执行

启动web服务器  `nohup bin/azkaban-web-start.sh 1>/tmp/azstd.out 2>/tmp/azerr.out &`

启动执行服务器 `bin/azkaban-web-start.sh`

## Demo

### 单一job

创建job描述文件

```bash
vim command.job

#command.job
type=command                                                    
command=echo 1111
```

打包文件

```
zip command.job
```

通过azkaban的web管理平台创建project并上传job压缩包
![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/5786888-324ed400a42fa942-1605454103-fd1664.webp)

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/5786888-98d49ae08853eec7-1605454132-1007a9.webp)





### 多job

1. 创建多个job

```bash
# foo.job
type=command
command=echo foo
```

```bash
# bar.job
type=command
dependencies=foo    # 表示依赖一个job
command=echo bar
```

2. 打包成zip
3. 上传执行

### 操作hadoop

```bash
# fs.job
type=command
command=hadoop fs -lsr /
```

### 操作hive

test.sql

```csharp
use default;
drop table aztest;
create table aztest(id int,name string,age int) row format delimited fields terminated by ',' ;
load data inpath '/aztest/hiveinput' into table aztest;
create table azres as select * from aztest;
insert overwrite directory '/aztest/hiveoutput' select count(1) from aztest; 
```

```bash
# hivef.job
type=command
command=hive -f 'test.sql'
```