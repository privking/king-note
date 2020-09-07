# hdfsShell命令
## 基本
* 在操作hdfs时 `hadoop fs <args>` 同 `hdfs dfs`
* `hadoop fs` 支持操作 Local FS, HFTP FS, S3 FS, and others.
* URI格式 `scheme://authority/path` HDFS的scheme为`hdfs`,本地文件系统的scheme为`file`

## appendToFile
 把文件追加到hadoop文件

##### usage

 usage: hadoop fs -appendToFile <localsrc> ... <dst>

##### demo

```
hadoop fs -appendToFile localfile /user/hadoop/hadoopfile
hadoop fs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
hadoop fs -appendToFile localfile hdfs://nn.example.com/hadoop/hadoopfile

//Reads the input from stdin.
hadoop fs -appendToFile - hdfs://nn.example.com/hadoop/hadoopfile 
```
##### return
* 0：success
* 1: error

## cat
将文件内容输出到stdout
##### usage
usage: hadoop fs -cat URI [URI ...]
##### demo
```
hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```
##### return
* 0：success
* -1:error

## checksum
返回被检查文件的格式信息
##### usage
usage: hadoop fs -checksum URI

##### demo
```
hadoop fs -checksum hdfs://nn1.example.com/file1
hadoop fs -checksum file:///etc/hosts
```


## chgrp
修改文件的group 用户必须为文件的所有者或者super-suer
##### usage
* -R 递归修改
```
 hadoop fs -chgrp [-R] GROUP URI [URI ...]
```

## chmod
修改文件权限
##### usage
* -R 递归修改
```
hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
```


## chown
修改文件的所有者,用户必须为super-user
##### usage

```
hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```


## copyFromLocal
类似于put命令将文件拷贝到本地文件系统
##### usage
* -f 覆盖已经存在的文件
```
 hadoop fs -copyFromLocal <localsrc> URI
```


## count
计算目录，文件，bytes的大小
##### usage
* -count 输出的列有  DIR_COUNT, FILE_COUNT, CONTENT_SIZE, PATHNAME
* -count -q 输出的列有 QUOTA, REMAINING_QUATA, SPACE_QUOTA, REMAINING_SPACE_QUOTA, DIR_COUNT, FILE_COUNT, CONTENT_SIZE, PATHNAME
* -h 将文件大小带上单位 human
* -v 展示表头
```
hadoop fs -count [-q] [-h] [-v] <paths>
```

##### demo

```
hadoop fs -count hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -count -q hdfs://nn1.example.com/file1
hadoop fs -count -q -h hdfs://nn1.example.com/file1
hdfs dfs -count -q -h -v hdfs://nn1.example.com/file1
```

##### return
* 0：success
* -1:error

## cp
拷贝文件
##### usage

```
hadoop fs -cp [-f] [-p | -p[topax]] URI [URI ...] <dest>
```

##### demo
* -f 覆盖已存在的文件
* -p 保持文件的属性（timestamps, ownership, permission, ACL, XAttr） 等不变 如果-p没有参数 则保持timestamps, ownership, permission不变
```
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
```

##### return
* 0：success
* -1:error

## df
显示空余大小
##### usage
* -h human-readable
```
 hadoop fs -df [-h] URI [URI ...]
```

##### demo
```
hadoop dfs -df /user/hadoop/dir1
```


## du 
展示目录空间使用情况

##### usage
* -s 显示文件长度的汇总摘要，而不是单个文件。
* -h human-readable
```
hadoop fs -du [-s] [-h] URI [URI ...]
```
##### demo
```
hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://nn.example.com/user/hadoop/dir1
```


## find
查找与指定表达式匹配的所有文件，并对其应用选定的操作。如果没有指定路径，则默认为当前工作目录。如果没有指定表达式，则默认为-print。
##### usage
* -name pattern
* -iname pattern 忽略大小写
* 多个条件 and
    * expression -a expression
    * expression -and expression
    * expression expression
```
hadoop fs -find <path> ... <expression> ...
```

##### demo

```
hadoop fs -find / -name test -print
```


## get
将文件拷贝到本地文件系统，CRC检查失败的文件可以用-ignorecrc选项复制。文件和CRCs可以复制使用-crc选项。

##### usage

```
hadoop fs -get [-ignorecrc] [-crc] <src> <localdst>
```

##### demo

```
hadoop fs -get /user/hadoop/file localfile
hadoop fs -get hdfs://nn.example.com/user/hadoop/file localfile
```
##### return
* 0：success
* -1:error


## getfacl
显示文件和目录的访问权限
##### usage
* -R: List the ACLs of all files and directories recursively.
```
hadoop fs -getfacl [-R] <path>
```

##### demo

```
hadoop fs -getfacl /file
hadoop fs -getfacl -R /dir
```

##### return
* 0：success
* -zero:error

## getmerge
合并文件 
##### usage
* addnl :add new line
```
hadoop fs -getmerge <src> <localdst> [addnl]
```

##### demo

```
hadoop fs -getmerge   /src  /opt/output.txt
hadoop fs -getmerge  /src/file1.txt /src/file2.txt  /output.txt
```


## ls
ls
##### usage
- -d: Directories are listed as plain files.
- -h: Format file sizes in a human-readable fashion (eg 64.0m instead of 67108864).
- -R: Recursively list subdirectories encountered.
- -t: Sort output by modification time (most recent first).
- -S: Sort output by file size.
- -r: Reverse the sort order.
- -u: Use access time rather than modification time for display and sorting.
- 
```
hadoop fs -ls [-d] [-h] [-R] [-t] [-S] [-r] [-u] <args>
```
##### demo
```
hadoop fs -ls /user/hadoop/file1
```

## lsr
递归ls
##### usage
* 过时usage 使用`hadoop fs -ls -R` 代替
```
 hadoop fs -lsr <args>
```


## mkdir
创建文件夹
##### usage
* -p 创建父文件夹
```
hadoop fs -mkdir [-p] <paths>
```

##### demo

```
hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
hadoop fs -mkdir hdfs://nn1.example.com/user/hadoop/dir hdfs://nn2.example.com/user/hadoop/dir
```

## moveFromLocal
* 类似于 `put`
##### usage

```
hadoop fs -moveFromLocal <localsrc> <dst>
```

## mv 
##### usage

```
hadoop fs -mv URI [URI ...] <dest>
```
##### demo

```
hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2
hadoop fs -mv hdfs://nn.example.com/file1 hdfs://nn.example.com/file2 hdfs://nn.example.com/file3 hdfs://nn.example.com/dir1
```

## put
##### usage

```
 hadoop fs -put <localsrc> ... <dst>
```
##### demo

```
hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put localfile hdfs://nn.example.com/hadoop/hadoopfile
hadoop fs -put - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin.
```

## rm
##### usage
- The -f option will not display a diagnostic message or modify the exit status to reflect an error if the file does not exist.
- The -R option deletes the directory and any content under it recursively.
- The -r option is equivalent to -R.
- The -skipTrash option will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.
```
hadoop fs -rm [-f] [-r |-R] [-skipTrash] URI [URI ...]
```
##### demo

```
hadoop fs -rm hdfs://nn.example.com/file /user/hadoop/emptydir
```
## rmdir
##### usage
```
hadoop fs -rmdir [--ignore-fail-on-non-empty] URI [URI ...]
```
##### demo

```
hadoop fs -rmdir /user/hadoop/emptydir
```
## tail
##### usage
* The -f option will output appended data as the file grows, as in Unix.
```
hadoop fs -tail [-f] URI
```

## test
##### usage
- -d: f the path is a directory, return 0.
- -e: if the path exists, return 0.
- -f: if the path is a file, return 0.
- -s: if the path is not empty, return 0.
- -z: if the file is zero length, return 0.

```
hadoop fs -test -[defsz] URI
```
## text
可以用来看压缩文件
##### usage

```
hadoop fs -text <src>
```
## touchz
创建空文件
##### usage

```
 hadoop fs -touchz URI [URI ...]
```
## usage
展示命令的用法
##### usage

```
hadoop fs -usage command
```