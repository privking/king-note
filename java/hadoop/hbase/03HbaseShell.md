# scHbaseShell

Hbase Shell中 分号表示换行

## help

help:查看命令帮助

help '组名' ：查看某个组的帮助

![image-20201031171234151](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031171234151-1604135561-69b2be.png)

![image-20201031171253158](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031171253158-1604135573-63b687.png)

help '命令' : 查看某个命令的帮助

## group general

### 显示regionserver task list

![image-20201031171941650](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031171941650-1604135981-5985d9.png)

### 显示 集群状态

![image-20201031172019475](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031172019475-1604136019-a0609d.png)

### 显示版本

![image-20201031172156066](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031172156066-1604136116-d208b8.png)

### 显示当前hbase用户

![image-20201031172223596](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201031172223596-1604136143-31e330.png)

## Group namespace

### alter_namespace

修改名称空间

![image-20201102010116864](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010116864-1604250076-86b258.png)

### create_namespace

创建名称空间

![image-20201102010034911](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010034911-1604250035-c4dcd4.png)

### describe_namespace

获取名称空间描述

![image-20201102010217063](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010217063-1604250137-67fdb1.png)

![image-20201102010230776](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010230776-1604250150-9f3ac0.png)

### drop_namespace

删除名称空间，名称空间必须为空

![image-20201102010253196](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010253196-1604250173-ed8def.png)

### list_namespace_tables

获取名称空间下 所有的表

![image-20201102010327604](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010327604-1604250207-276c86.png)

### list_namespace

列出所有的namesoace

![image-20201102005907522](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102005907522-1604249947-df1af4.png)

## Group ddl

### alter

修改之前必须设置为disable状态

可以增删改列族，参数

![image-20201102010651631](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102010651631-1604250411-4683a7.png)

修改 table-scope 参数

![image-20201102011329367](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102011329367-1604250809-83dcbc.png)

### alter_async

异步的alter 不会等到所有的regions都收到改变的消息

![image-20201102011645471](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102011645471-1604251005-5d7ee6.png)

### alter_status

获取alter的命令执行进度

![image-20201102012354189](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102012354189-1604251434-92ef08.png)

### clone_table_schema

 Passing 'false' as the optional third parameter will not preserve split keys

![image-20201102012516601](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102012516601-1604251516-70e14a.png)

### create

创建表，通过表名和列族的属性参数等的集合，必须包含 NAME属性

![image-20201102012704237](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102012704237-1604251624-e93c3a.png)

### describe

可以用desc 代替

![image-20201102013159948](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013159948-1604251920-4208d6.png)

### disable & disable_all

![image-20201102013248366](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013248366-1604251968-49b966.png)

### drop & drop_all

![image-20201102013308362](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013308362-1604251988-9415f1.png)

### enable & enable_all

![image-20201102013329066](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013329066-1604252009-7cfe7e.png)

### exists

![image-20201102013359452](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013359452-1604252039-bf9f76.png)

### get_table

取别名


![image-20201102013447649](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013447649-1604252087-d6a273.png)

### is_disable & is_enable

![image-20201102013643902](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013643902-1604252203-b0bf51.png)

### list

获取所有表

![image-20201102013725362](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013725362-1604252245-88dcc8.png)

### list_regions

列出符合条件的regions

![image-20201102013831973](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013831973-1604252312-d53616.png)

### show_filters

![image-20201102013914363](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102013914363-1604252354-0fffa3.png)

## Group dml

### append

![image-20201102014702439](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102014702439-1604252822-88a59d.png)

### count

![image-20201102014751966](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102014751966-1604252872-0c919c.png)

### delete &delete_all

![image-20201102014842061](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102014842061-1604252922-38e369.png)

### get

![image-20201102014935372](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102014935372-1604252975-6df3f8.png)

### get_conter

![image-20201102015031396](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015031396-1604253031-eb67f7.png)

### get_splits

![image-20201102015054025](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015054025-1604253054-ecce0a.png)

### incr

![image-20201102015113882](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015113882-1604253073-a85d58.png)

### put

![image-20201102015206032](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015206032-1604253126-578c4e.png)

![image-20201102021210332](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102021210332-1604254330-be2331.png)

### scan

参数：COLUMNS-> 扫描指定列

LIMIT->取前n行

STARTROW->从哪一行开始（包含）

STOPROW->=结束位置 （不包含）

ROW=true 显示原始行 包括是否是删除类型等



![image-20201102015306333](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015306333-1604253186-f665eb.png)

![image-20201102015330330](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015330330-1604253210-f3fa77.png)

### truncate & truncate_preserve

![image-20201102015509437](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102015509437-1604253309-1b0fc8.png)