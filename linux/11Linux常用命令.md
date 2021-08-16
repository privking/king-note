# Linux常用命令

## ls

可以查看 linux 文件夹包含的文件or文件权限(包括目录、文件夹、文件权限)查看目录信息等等

**常用参数**

- -l ：列出长数据串，包含文件的属性与权限数据等
- -a ：列出全部的文件，连同隐藏文件（开头为.的文件）一起列出来（常用）
- -d ：仅列出目录本身，而不是列出目录的文件数据
- -h ：将文件容量以较易读的方式（GB，kB等）列出来
- -R ：连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来

**example**

```shell
ls -al /home
```

## cd

用于切换当前目录至dirName

**example**

```sh
cd /home/
cd ~ #用户家目录
cd - #最近使用目录
```

## pwd

查看"当前工作目录"的完整路径。

**常用参数**

- -P :显示实际物理路径，而非使用连接（link）路径
- -L :当目录为连接路径时，显示连接路径

## mkdir 

用来创建指定的名称的目录，要求创建目录的用户在当前目录中具有写权限，并且指定的目录名不能是当前目录中已有的目录

**常用参数**

- -m, --mode=模式，设定权限<模式> (类似 chmod)，而不是 rwxrwxrwx 减 umask
- -p, --parents 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后,系统将自动建立好那些尚不存在的目录,即一次可以建立多个目录;
- -v, --verbose 每次创建新目录都显示信息
- --help 显示此帮助信息并退出
- --version 输出版本信息并退出

**example**

```sh
mkdir -m 777 test2
mkdir -p demo/demo1/demo2
```

## rm

删除一个目录中的一个或多个文件或目录

**常用参数**

- -f, --force 忽略不存在的文件，从不给出提示。
- -i, --interactive 进行交互式删除
- -r, -R, --recursive 指示rm将参数中列出的全部目录和子目录均递归地删除。
- -v, --verbose 详细显示进行的步骤
- --help 显示此帮助信息并退出
- --version 输出版本信息并退出

**example**

```sh
rm -rf test
```

## mv

可以用来移动文件或者将文件改名（move (rename) files）。当第二个参数类型是文件时，mv命令完成文件重命名。当第二个参数是已存在的目录名称时，源文件或目录参数可以有多个，mv命令将各参数指定的源文件均移至目标目录中。

**常用参数**

- -b ：若需覆盖文件，则覆盖前先行备份
- -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖
- -i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖
- -u ：若目标文件已经存在，且 source 比较新，才会更新(update)
- -t ： --target-directory=DIRECTORY move all SOURCE arguments into DIRECTORY，即指定mv的目标目录，该选项适用于移动多个源文件到一个目录的情况，此时目标目录在前，源文件在后

**example**

```sh
mv test1.txt test2
```

## cp

将源文件复制至目标文件，或将多个源文件复制至目标目录。

**常用参数**

- -t --target-directory 指定目标目录
- -i --interactive 覆盖前询问（使前面的 -n 选项失效）
- -n --no-clobber 不要覆盖已存在的文件（使前面的 -i 选项失效）
- -f --force 强行复制文件或目录，不论目的文件或目录是否已经存在
- -u --update 使用这项参数之后，只会在源文件的修改时间较目的文件更新时，或是对应的目的文件并不存在，才复制文件

**example**

```sh
cp test1.txt test1 # 若文件存在，会提示是否覆盖。若不存在直接完成复制
```

## touch 

touch命令参数可更改文档或目录的日期时间，包括存取时间和更改时间。

**常用参数**

- -a 或--time=atime或--time=access或--time=use 　只更改存取时间
- -c 或--no-create 　不建立任何文档
- -d 　使用指定的日期时间，而非现在的时间
- -f 　此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题
- -m 或--time=mtime或--time=modify 　只更改变动时间
- -r 　把指定文档或目录的日期时间，统统设成和参考文档或目录的日期时间相同 -t 　使用指定的日期时间，而非现在的时间

**example**

```sh
#创建不存在的文件test.txt
touch test.txt
#更新 test.txt 的实践和 test1.txt 时间戳相同
touch -r test.txt test1.txt
```

## cat

用来显示文件内容，或者将几个文件连接起来显示，或者从标准输入读取内容并显示，它常与重定向符号配合使用。

**常用参数**

- -A, --show-all 等价于 -vET
- -b, --number-nonblank 对非空输出行编号
- -e 等价于 -vE
- -E, --show-ends 在每行结束处显示 $
- -n, --number 对输出的所有行编号,由1开始对所有输出的行数编号
- -s, --squeeze-blank 有连续两行以上的空白行，就代换为一行的空白行
- -t 与 -vT 等价
- -T, --show-tabs 将跳格字符显示为 ^I
- -u (被忽略)
- -v, --show-nonprinting 使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外

**example**

```sh
#把 test.log 的文件内容加上行号后输入 test1.log 这个文件里。
cat -n test.log test1.log
#将 test.log 的文件内容反向显示。
tac test.log

:~# cat c.txt
:~# cat >c.txt <<EOF
> 1111111
> 2222222
> EOF
:~# cat c.txt
1111111
2222222

:~# cat >c.txt <<EOF
> 33333333
> EOF
:~# cat c.txt
33333333
:~#
```

## nl

输出的文件内容自动的加上行号！其默认的结果与 cat -n 有点不太一样， nl 可以将行号做比较多的显示设计，包括位数与是否自动补齐 0 等等的功能。

**常用参数**

- -b ：指定行号指定的方式，主要有两种：
  - -b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)
  - -b t ：如果有空行，空的那一行不要列出行号(默认值)
- -n ：列出行号表示的方法，主要有三种：
  - -n ln ：行号在萤幕的最左方显示
  - -n rn ：行号在自己栏位的最右方显示，且不加 0
  - -n rz ：行号在自己栏位的最右方显示，且加 0
- -w ：行号栏位的占用的位数

**example**

```sh
nl -b a test.log

nl -bt -nrn -w 10  hello.txt
```

## more

more 命令和 cat 的功能一样都是查看文件里的内容，但有所不同的是more可以按页来查看文件的内容，还支持直接跳转行等功能。

**常用参数**

- +n 从笫n行开始显示
- -n 定义屏幕大小为n行
- +/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示
- -c 从顶部清屏，然后显示
- -d 提示“Press space to continue，’q’ to quit（按空格键继续，按q键退出）”，禁用响铃功能
- -l 忽略Ctrl+l（换页）字符
- -p 通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似
- -s 把连续的多个空行显示为一行
- -u 把文件内容中的下画线去掉

**操作指令**

- Enter：向下n行，需要定义。默认为1行
- Ctrl+F：向下滚动一屏
- 空格键：向下滚动一屏
- Ctrl+B：返回上一屏
- = ：输出当前行的行号
- ：f ：输出文件名和当前行的行号
- V ：调用vi编辑器
- !命令 ：调用Shell，并执行命令
- q ：退出more

**example**

```sh
#显示文件 test.log 第3行起内容
more +3 test.log
#从文件 test.log 查找第一个出现“day3”字符串的行，并从该处前2行开始显示输出。
more +/day3 test.log
#设置每屏显示行数
more -5 test.log
```

## less

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件

**常用参数**

- -b <缓冲区大小> 设置缓冲区的大小
- -e 当文件显示结束后，自动离开
- -f 强迫打开特殊文件，例如外围设备代号、目录和二进制文件
- -g 只标志最后搜索的关键词
- -i 忽略搜索时的大小写
- -m 显示类似more命令的百分比
- -N 显示每行的行号
- -o <文件名> 将less 输出的内容在指定文件中保存起来
- -Q 不使用警告音
- -s 显示连续空行为一行
- -S 行过长时间将超出部分舍弃
- -x <数字> 将“tab”键显示为规定的数字空格

**操作指令**

- /字符串：向下搜索“字符串”的功能
- ?字符串：向上搜索“字符串”的功能
- n：重复前一个搜索（与 / 或 ? 有关）
- N：反向重复前一个搜索（与 / 或 ? 有关）
- b 向后翻一页
- d 向后翻半页
- h 显示帮助界面
- Q 退出less 命令
- u 向前滚动半页
- y 向前滚动一行
- 空格键 滚动一页
- 回车键 滚动一行
- [pagedown]： 向下翻动一页
- [pageup]： 向上翻动一页

**example**

```sh
less test.log
```



## head

head 用来显示档案的开头至标准输出中，默认 head 命令打印其相应文件的开头 10 行。

**常用参数**

- -q 隐藏文件名
- -v 显示文件名
- -c<字节> 显示字节数
- -n<行数> 显示的行数

**example**

```sh
head -n 5 test.log
```



## tail 

显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。

**常用参数**

- -f 循环读取
- -q 不显示处理信息
- -v 显示详细的处理信息
- -c<数目> 显示的字节数
- -n<行数> 显示行数
- --pid=PID 与-f合用,表示在进程ID,PID死掉之后结束.
- -q, --quiet, --silent 从不输出给出文件名的首部
- -s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒

**example**

```sh
tail -f -n 5 test.log
```



## which

which指令会在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。

**example**

```sh
which pwd
which which
```

## whereis

**常用参数**

- -b 定位可执行文件
- -m 定位帮助文件
- -s 定位源代码文件
- -u 搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件
- -B 指定搜索可执行文件的路径
- -M 指定搜索帮助文件的路径
- -S 指定搜索源代码文件的路径

**example**

```sh
whereis git
```

## find

指定目录下查找文件

如果 path 是空字串则使用目前路径，如果 expression 是空字串则使用 -print 为预设 expression

`find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;`

**常用参数**

- -mount, -xdev : 只检查和指定目录在同一个文件系统下的文件，避免列出其它文件系统中的文件
- -amin n : 在过去 n 分钟内被读取过
- -anewer file : 比文件 file 更晚被读取过的文件
- -atime n : 在过去n天内被读取过的文件
- -cmin n : 在过去 n 分钟内被修改过
- -cnewer file :比文件 file 更新的文件
- -ctime n : 在过去n天内被修改过的文件
- -empty : 空的文件-gid n or -group name : gid 是 n 或是 group 名称是 name
- -ipath p, -path p : 路径名称符合 p 的文件，ipath 会忽略大小写
- -name name, -iname name : 文件名称符合 name 的文件。iname 会忽略大小写
- -size n : 文件大小 是 n 单位，b 代表 512 位元组的区块，c 表示字元数，k 表示 kilo bytes，w 是二个位元组。
- -type c : 文件类型是 c 的文件。
  - d: 目录
  - c: 字型装置文件
  - b: 区块装置文件
  - p: 具名贮列
  - f: 一般文件
  - l: 符号连结

**example**

```sh
find . -name "*.c"
find . -type f
find . -ctime -20
find /var/log -type f -mtime +7 -ok rm {} \;
find . -type f -perm 644 -exec ls -l {} \;
```



## tar

解压缩

**常用参数**

- -A 新增压缩文件到已存在的压缩
- -B 设置区块大小
- -c 建立新的压缩文件
- -d 记录文件的差别
- -r 添加文件到已经压缩的文件
- -u 添加改变了和现有的文件到已经存在的压缩文件
- -x 从压缩的文件中提取文件
- -t 显示压缩文件的内容
- -z 支持gzip解压文件
- -j 支持bzip2解压文件
- -Z 支持compress解压文件
- -v 显示操作过程
- -l 文件系统边界设置
- -k 保留原有文件不覆盖
- -m 保留文件不被覆盖
- -W 确认压缩文件的正确性
- -b 设置区块数目
- -C 切换到指定目录
- -f 指定压缩文件
- --help 显示帮助信息
- --version 显示版本信息

```sh
tar -zxvf test.tar.gz   #解压

tar -zcvf test.tar.gz test.log  # 打包后，以 gzip 压缩 

tar -zcvf test.tar.bz2 test.log # 打包后，以 bzip2 压缩
```



## chmod

用于改变linux系统文件或目录的访问权限

**常用参数**

- -c 当发生改变时，报告处理信息
- -f 错误信息不输出
- -R 处理指定目录以及其子目录下的所有文件
- -v 运行时显示详细处理信息
- 选择参数：
- --reference=<目录或者文件> 设置成具有指定目录或者文件具有相同的权限
- --version 显示版本信息
- <权限范围>+<权限设置> 使权限范围内的目录或者文件具有指定的权限
- <权限范围>-<权限设置> 删除权限范围的目录或者文件的指定权限
- <权限范围>=<权限设置> 设置权限范围内的目录或者文件的权限为指定的值

**权限范围**

- u ：目录或者文件的当前的用户
- g ：目录或者文件的当前的群组
- o ：除了目录或者文件的当前用户或群组之外的用户或者群组
- a ：所有的用户及群组

**权限代号**

- r：读权限，用数字4表示
- w：写权限，用数字2表示
- x：执行权限，用数字1表示
- -：删除权限，用数字0表示

**example**

```sh
chmod a+x test.log
chmod 777 test.log
```

## chgrp

修改组

**常用参数**

- -c 当发生改变时输出调试信息
- -f 不显示错误信息
- -R 处理指定目录以及其子目录下的所有文件
- -v 运行时显示详细的处理信息
- --dereference 作用于符号链接的指向，而不是符号链接本身
- --no-dereference 作用于符号链接本身

**example**

```sh
chgrp -v bin test.log # 修改文件所属组为bin
```



## chown 

改变文件的拥有者和群组

**常用参数**

- -c 显示更改的部分的信息
- -f 忽略错误信息
- -h 修复符号链接
- -R 处理指定目录以及其子目录下的所有文件
- -v 显示详细的处理信息
- -deference 作用于符号链接的指向，而不是链接文件本身

**example**

```sh
chown mail:mail test.log
```



## df

显示指定磁盘文件的可用空间

**常用参数**

- -a 全部文件系统列表
- -h 方便阅读方式显示
- -H 等于“-h”，但是计算式，1K=1000，而不是1K=1024
- -i 显示inode信息
- -k 区块为1024字节
- -l 只显示本地文件系统
- -m 区块为1048576字节
- --no-sync 忽略 sync 命令
- -P 输出格式为POSIX
- --sync 在取得磁盘信息前，先执行sync命令
- -T 文件系统类型

**example**

```sh
df -hTt ext4
```



## du

显示每个文件和目录的磁盘使用空间

**常用参数**

- -a或-all 显示目录中个别文件的大小。
- -b或-bytes 显示目录或文件大小时，以byte为单位。
  -- -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
- -k或--kilobytes 以KB(1024bytes)为单位输出。
- -m或--megabytes 以MB为单位输出。
- -s或--summarize 仅显示总计，只列出最后加总的值。
- -h或--human-readable 以K，M，G为单位，提高信息的可读性。
- -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
- -L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。
- -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
- -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
- --exclude=<目录或文件> 略过指定的目录或文件。
- -D或--dereference-args 显示指定符号链接的源文件大小。
- -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
- -l或--count-links 重复计算硬件链接的文件。

**example**

```sh
du -hs ./  # 显示./目录占用大小
du -hs ./* # 小时./下一级子目录分别占用大小
```

## top

显示当前系统正在执行的进程的相关信息，包括进程ID、内存占用率、CPU占用率等

**常用参数**

- -b 批处理
- -c 显示完整的治命令
- -I 忽略失效过程
- -s 保密模式
- -S 累积模式
- -i<时间> 设置间隔时间
- -u<用户名> 指定用户名
- -p<进程号> 指定进程
- -n<次数> 循环显示的次数

**example**

```sh
top -cp 779 #查看779号进程使用情况
top 
```



## free

显示系统使用和空闲的内存情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。

**常用参数**

- -b 　以Byte为单位显示内存使用情况
- -k 　以KB为单位显示内存使用情况
- -m 　以MB为单位显示内存使用情况
- -g 以GB为单位显示内存使用情况
- -o 　不显示缓冲区调节列
- -s<间隔秒数> 　持续观察内存使用状况
- -t 　显示内存总和列。
- -V 　显示版本信息。

**example**

```sh
free
free -g #以GB为单位
free -m #以MB为单位
```

## ifconfig

**常用参数**

- up 启动指定网络设备/网卡
- down 关闭指定网络设备/网卡。
- arp 设置指定网卡是否支持ARP协议
- -promisc 设置是否支持网卡的promiscuous模式，如果选择此参数，网卡将接收网络中发给它所有的数据包
- -allmulti 设置是否支持多播模式，如果选择此参数，网卡将接收网络中所有的多播数据包
- -a 显示全部接口信息
- -s 显示摘要信息（类似于 netstat -i）
- add 给指定网卡配置IPv6地址
- del 删除指定网卡的IPv6地址

**example**

```sh
ifconfig eth0 up
ifconfig eth0 down
```

## route 

Route命令是用于操作基于内核ip路由表，

**常用参数**

- -c 显示更多信息
- -n 不解析名字
- -v 显示详细的处理信息
- -F 显示发送信息
- -C 显示路由缓存
- -f 清除所有网关入口的路由表。
- -p 与 add 命令一起使用时使路由具有永久性。
- add:添加一条新路由。
- del:删除一条路由。
- -net:目标地址是一个网络。
- -host:目标地址是一个主机。
- netmask:当添加一个网络路由时，需要使用网络掩码。
- gw:路由数据包通过网关。注意，你指定的网关必须能够达到。
- metric：设置路由跳数。
- Command 指定您想运行的命令 (Add/Change/Delete/Print)。
- Destination 指定该路由的网络目标。

**example**

```sh
route 
route -n
route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0
```



## ping

确定网络和各外部主机的状态

**常用参数**

- -d 使用Socket的SO_DEBUG功能
- -f 极限检测。大量且快速地送网络封包给一台机器，看它的回应
- -n 只输出数值
- -q 不显示任何传送封包的信息，只显示最后的结果
- -r 忽略普通的Routing Table，直接将数据包送到远端主机上。通常是查看本机的网络接口是否有问题
- -R 记录路由过程
- -v 详细显示指令的执行过程
- -c 数目：在发送指定数目的包后停止
- -i 秒数：设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次 -I 网络界面：使用指定的网络界面送出数据包 -l 前置载入：设置在送出要求信息之前，先行发出的数据包 -p 范本样式：设置填满数据包的范本样式 -s 字节数：指定发送的数据字节数，预设值是56，加上8字节的ICMP头，一共是64ICMP数据字节 -t 存活数值：设置存活数值TTL的大小

**example**

```sh
ping baidu.com
ping -R baidu.com
```



## ps

用来显示当前进程的状态

- a 显示所有进程
- -a 显示同一终端下的所有程序
- -A 显示所有进程
- c 显示进程的真实名称
- -N 反向选择
- -e 等于“-A”
- e 显示环境变量
- f 显示程序间的关系
- -H 显示树状结构
- r 显示当前终端的进程
- T 显示当前终端的所有程序
- u 指定用户的所有进程
- -au 显示较详细的资讯
- -aux 显示所有包含其他使用者的行程
- -C<命令> 列出指定命令的状况
- --lines<行数> 每页显示的行数
- --width<字符数> 每页显示的字符数

**example**

```sh
ps -ef

ps -aux
```



## netstat 

用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。

**常用参数**

- -a或–all 显示所有连线中的Socket
- -A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址
- -c或–continuous 持续列出网络状态
- -C或–cache 显示路由器配置的快取信息
- -e或–extend 显示网络其他相关信息
- -F或–fib 显示FIB
- -g或–groups 显示多重广播功能群组组员名单
- -h或–help 在线帮助
- -i或–interfaces 显示网络界面信息表单
- -l或–listening 显示监控中的服务器的Socket
- -M或–masquerade 显示伪装的网络连线
- -n或–numeric 直接使用IP地址，而不通过域名服务器
- -N或–netlink或–symbolic 显示网络硬件外围设备的符号连接名称
- -o或–timers 显示计时器
- -p或–programs 显示正在使用Socket的程序识别码和程序名称
- -r或–route 显示Routing Table
- -s或–statistice 显示网络工作信息统计表
- -t或–tcp 显示TCP传输协议的连线状况
- -u或–udp 显示UDP传输协议的连线状况
- -v或–verbose 显示指令执行过程
- -V或–version 显示版本信息
- -w或–raw 显示RAW传输协议的连线状况
- -x或–unix 此参数的效果和指定”-A unix”参数相同
- –ip或–inet 此参数的效果和指定”-A inet”参数相同

**example**

```sh
netstat -lnpt
```



## ln

为某一个文件在另外一个位置建立一个同步的链接.当我们需要在不同的目录，用到相同的文件时，我们不需要在每一个需要的目录下都放一个必须相同的文件，我们只要在某个固定的目录，放上该文件，然后在 其它的目录下用ln命令链接（link）它就可以，不必重复的占用磁盘空间。

硬链接和源文件是同一份文件，而软连接是独立的文件，类似于快捷方式

不能对目录创建硬链接，不能对不同文件系统创建硬链接，不能对不存在的文件创建硬链接；

可以对目录创建软连接，可以跨文件系统创建软连接，可以对不存在的文件创建软连接。

**只有删除了源文件和所有对应的硬链接文件，文件实体才会被删除**



**常用参数**

- -b 删除，覆盖以前建立的链接
- -d 允许超级用户制作目录的硬链接
- -f 强制执行
- -i 交互模式，文件存在则提示用户是否覆盖
- -n 把符号链接视为一般目录
- -s 软链接(符号链接)
- -v 显示详细的处理过程

**example**

```sh
ln -s test.log linktest # 为 test.log文件创建软链接linktest
ln test.log lntest #为 test.log创建硬链接lntest
```



## diff

比较文件或者目录内容

**常用参数**

- -c 上下文模式，显示全部内文，并标出不同之处
- -u 统一模式，以合并的方式来显示文件内容的不同
- -a 只会逐行比较文本文件
- -N 在比较目录时，若文件 A 仅出现在某个目录中，预设会显示：Only in 目录。若使用 -N 参数，则 diff 会将文件 A 与一个空白的文件比较
- -r 递归比较目录下的文件

**example**

```sh
diff test1.txt test2.txt
```



## grep

文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来

**常用参数**

- -c 计算找到‘搜寻字符串’（即 pattern）的次数
- -i 忽略大小写的不同，所以大小写视为相同
- -n 输出行号
- -v 反向选择，打印不匹配的行
- -r 递归搜索
- --color=auto 将找到的关键词部分加上颜色显示

**example**

```sh
grep -n "hello" ./hello.txt

cat hello.txt | grep "hello"
```



## wc

用来显示文件所包含的行、字和字节数。

**常用参数**

- -c 统计字节数
- -l 统计行数
- -m 统计字符数，这个标志不能与 -c 标志一起使用
- -w 统计字数，一个字被定义为由空白、跳格或换行字符分隔的字符串
- -L 打印最长行的长度

**example**

```sh
wc -c test.txt
wc -l test.txt
wc -m test.txt

cat test.txt | wc -c
cat test.txt | wc -l
cat test.txt | wc -m
```

## ssh

远程登录

**常用命令**

- -4 #强制ssh协议只使用IPv4地址
- -6 #强制ssh协议只使用IPv6地址
- -A #启用来自身份验证代理的连接转发
- -a  #禁用身份验证代理连接的转发
- -B bind_interface  #绑定到的地址 bind_interface在尝试连接到目标主机之前
- -b bind_address    #使用 bind_address在本地计算机上作为连接的源地址
- -C  #请求压缩所有数据
- -c cipher_spec  #指定用于加密会话的密码规范
- -D [bind_address：] 端口  #指定本地“动态”应用程序级端口转发
- -E log_file   #将调试日志附加到 log_file 而非标准错误
- -e escape_char  #设置带有pty的会话的转义字符（默认值：' ~'）
- -F 配置文件  #指定每用户ssh的配置文件 
- -f  #配置ssh在执行命令之前将请求转到后台 
- -g  #允许远程主机连接到本地转发的端口
- -i identity_file  #指定从这个文件中去读取用于公共密钥身份验证的标识（私有密钥）
- -K  #启用基于GSSAPI的身份验证
- -k  #禁用将GSSAPI凭据
- -L local_socket：remote_socket  #指定将与本地（客户端）主机上给定的TCP端口或Unix套接字的连接转发到远程的给定主机和端口或Unix套接字。
- -N  #禁止执行远程命令 
- -p port  #指定SSH连接端口
- -q  #静默模式
- -s  #用于请求调用远程系统上的子系统
- -T  #禁用分配伪终端
- -t  #强制分配伪终端
- -V  #打印SSH版本号并退出
- -v  #详细模式（输出SSH连接的过程信息）
- -X  #启用X11转发
- -x  #禁用X11转发
- -Y  #启用受信任的X11转发
- -y  #指定发送日志信息 syslog（3）系统模块

**example**

```sh
ssh username@remote_host -p 9999
```

## scp

远程拷贝

**常用参数**

- -1： 强制scp命令使用协议ssh1
- -2： 强制scp命令使用协议ssh2
- -4： 强制scp命令只使用IPv4寻址
- -6： 强制scp命令只使用IPv6寻址
- -B： 使用批处理模式（传输过程中不询问传输口令或短语）
- -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
- -p：保留原文件的修改时间，访问时间和访问权限。
- -q： 不显示传输进度条。
- -r： 递归复制整个目录。
- -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
- -c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
- -F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
- -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
- -l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
- -o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
- -P port：注意是大写的P, port是指定数据传输用到的端口号
- -S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。



**example**

```sh
scp local_file remote_username@remote_ip:remote_folder 

scp local_file remote_username@remote_ip:remote_file 

scp local_file remote_ip:remote_folder 

scp local_file remote_ip:remote_file 
```

## nc

netcat 用于设置路由器

**常用参数**

```
 -4                         Use IPv4 only
  -6                         Use IPv6 only
  -U, --unixsock             Use Unix domain sockets only
  -C, --crlf                 Use CRLF for EOL sequence
  -c, --sh-exec <command>    Executes the given command via /bin/sh
  -e, --exec <command>       Executes the given command
      --lua-exec <filename>  Executes the given Lua script
  -g hop1[,hop2,...]         Loose source routing hop points (8 max)
  -G <n>                     Loose source routing hop pointer (4, 8, 12, ...)
  -m, --max-conns <n>        Maximum <n> simultaneous connections
  -h, --help                 Display this help screen
  -d, --delay <time>         Wait between read/writes
  -o, --output <filename>    Dump session data to a file
  -x, --hex-dump <filename>  Dump session data as hex to a file
  -i, --idle-timeout <time>  Idle read/write timeout
  -p, --source-port port     Specify source port to use
  -s, --source addr          Specify source address to use (doesn't affect -l)
  -l, --listen               Bind and listen for incoming connections
  -k, --keep-open            Accept multiple connections in listen mode
  -n, --nodns                Do not resolve hostnames via DNS
  -t, --telnet               Answer Telnet negotiations
  -u, --udp                  Use UDP instead of default TCP
      --sctp                 Use SCTP instead of default TCP
  -v, --verbose              Set verbosity level (can be used several times)
  -w, --wait <time>          Connect timeout
  -z                         Zero-I/O mode, report connection status only
      --append-output        Append rather than clobber specified output files
      --send-only            Only send data, ignoring received; quit on EOF
      --recv-only            Only receive data, never send anything
      --allow                Allow only given hosts to connect to Ncat
      --allowfile            A file of hosts allowed to connect to Ncat
      --deny                 Deny given hosts from connecting to Ncat
      --denyfile             A file of hosts denied from connecting to Ncat
      --broker               Enable Ncat's connection brokering mode
      --chat                 Start a simple Ncat chat server
      --proxy <addr[:port]>  Specify address of host to proxy through
      --proxy-type <type>    Specify proxy type ("http" or "socks4" or "socks5")
      --proxy-auth <auth>    Authenticate with HTTP or SOCKS proxy server
      --ssl                  Connect or listen with SSL
      --ssl-cert             Specify SSL certificate file (PEM) for listening
      --ssl-key              Specify SSL private key (PEM) for listening
      --ssl-verify           Verify trust and domain name of certificates
      --ssl-trustfile        PEM file containing trusted SSL certificates
      --ssl-ciphers          Cipherlist containing SSL ciphers to use
      --version              Display Ncat's version information and exit
```

**example**

```sh
nc -lk 9999
```

## echo

输出

**常用参数**

- -n 不要在最后自动换行
- -e 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般



-e解析

-  \a 发出警告声；
-   \b 删除前一个字符；
-   \c 最后不加上换行符号；
-   \f 换行但光标仍旧停留在原来的位置；
-   \n 换行且光标移至行首；
-   \r 光标移至行首，但不换行；
-   \t 插入tab；
-   \v 与\f相同；
-   \\ 插入\字符；
-   \nnn 插入nnn（八进制）所代表的ASCII字符；

**example**

```sh
echo -e "hello\bi" # helli
```

## cut

对数据列进行提取

**常用参数**

- -d 指定分割符
- -f  指定截取区域
- -c  以字符为单位进行分割

**example**

```sh
# -d选项，默认为制表符
#以':'为分隔符，截取出/etc/passwd的第一列跟第三列
cut -d ':' -f 1,3 /etc/passwd
#以':'为分隔符，截取出/etc/passwd的第一列到第三列
cut -d ':' -f 1-3 /etc/passwd
#以':'为分隔符，截取出/etc/passwd的第二列到最后一列
cut -d ':' -f 2- /etc/passwd
#截取/etc/passwd文件从第二个字符到第九个字符
cut -c 2-9 /etc/passwd
```

## awk

对数据列进行提取

- `awk '条件 {执行动作}'文件名`
- `awk '条件1 {执行动作} 条件2 {执行动作} ...' 文件名`
- `awk [选项] '条件1 {执行动作} 条件2 {执行动作} ...' 文件名`

**常用参数**

- -F 指定分割符
- BEGIN 在读取所有行内容前就开始执行，常常被用于修改内置变量的值
- END 结束的时候 执行
- NF                 浏览记录的域的个数
- NR                 行号
- ARGC               命令行参数个数
- ARGV               命令行参数排列
- ENVIRON            支持队列中系统环境变量的使用
- FILENAME           awk浏览的文件名
- FNR                浏览文件的记录数
- FS                 设置输入域分隔符，等价于命令行 -F选项
- OFS                输出域分隔符
- ORS                输出记录分隔符
- RS                 控制记录分隔符

**example**

```sh
# $1 		#代表第一列
# $2 		#代表第二列
# $0 		#代表一整行
cat /etc/passwd | awk -F":" '{print $1}'

#BEGIN时定义分割符
cat /etc/passwd | awk 'BEGIN {FS=":"} {print $1}'

cat /etc/passwd | awk -F":" '{print $1} END {printf "以上为执行结果\n"}'
# 输出第二行第五个
df -h | awk 'NR==2 {print $5}'
# 输出20-30行第一个
awk '(NR>=20 && NR<=30) {print $1}' /etc/passwd


#awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
#filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/bin/bash
#filename:/etc/passwd,linenumber:2,columns:7,linecontent:daemon:x:1:1:daemon:/usr/sbin:/bin/sh
#filename:/etc/passwd,linenumber:3,columns:7,linecontent:bin:x:2:2:bin:/bin:/bin/sh
#filename:/etc/passwd,linenumber:4,columns:7,linecontent:sys:x:3:3:sys:/dev:/bin/sh
```

## read

读取数据并赋值给变量

**常用参数**

- -d :指定读取结束位置，默认换行符
- -n num :读取num个字符，读取够以后就不用等到结束位置
- -p prompt:显示提示信息
- -r:不转义
- -s:静默模式，不显示输入的字符
- -t seconds:设置超时时间
- -u fd :使用文件描述符作为输入

**example**

```sh
if
    read -t 20 -sp "Enter password in 20 seconds(once) > " pass1 && printf "\n" &&  #第一次输入密码
    read -t 20 -sp "Enter password in 20 seconds(again)> " pass2 && printf "\n" &&  #第二次输入密码
    [ $pass1 == $pass2 ]  #判断两次输入的密码是否相等
then
    echo "Valid password"
else
    echo "Invalid password"
fi
```

```
如果两次输入密码相同，运行结果为：
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)>
Valid password

如果两次输入密码不同，运行结果为：
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)>
Invalid password

如果第一次输入超时，运行结果为：
Enter password in 20 seconds(once) > Invalid password

如果第二次输入超时，运行结果为：
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)> Invalid password
```

## date

日期命令

**常用参数**

- -d<字符串>：显示字符串所指的日期与时间。字符串前后必须加上双引号； 
- -s<字符串>：根据字符串来设置日期与时间。字符串前后必须加上双引号；
-  -u：显示GMT； 
- --help：在线帮助；
-  --version：显示版本信息。

**格式化字符串**

- **%Y 年份(以四位数来表示)。** 
- **%m 月份(以01-12来表示)。** 
- **%d 日期(以01-31来表示)。** 
- **%H 小时(以00-23来表示)。** 
- **%M 分钟(以00-59来表示)。**
-  **%S 秒(以本地的惯用法来表示)。** 
- %I 小时(以01-12来表示)。
-  %K 小时(以0-23来表示)。 
- %l 小时(以0-12来表示)。 
-  %P AM或PM。 
- %r 时间(含时分秒，小时以12小时AM/PM来表示)。 
- %s 总秒数。起算时间为1970-01-01 00:00:00 UTC。
- %T 时间(含时分秒，小时以24小时制来表示)。 
- %X 时间(以本地的惯用法来表示)。
-  %Z 市区。 
- %a 星期的缩写。 
- %A 星期的完整名称。
-  %b 月份英文名的缩写。 
- %B 月份的完整英文名称。 
- %c 日期与时间。只输入date指令也会显示同样的结果。 
- %D 日期(含年月日)。 
- %j 该年中的第几天。 
- %U 该年中的周数。 
- %w 该周的天数，0代表周日，1代表周一，异词类推。 
- %x 日期(以本地的惯用法来表示)。 
- %y 年份(以00-99来表示)。 
- %n 在显示时，插入新的一行。 
- %t 在显示时，插入tab。 
- MM 月份(必要) 
- DD 日期(必要) 
- hh 小时(必要) 
- mm 分钟(必要)
- ss 秒(选择性)

**example**

```shell
date +"%Y-%m-%d"  
#2015-12-07
date -d "Dec 5, 2009 12:00:37 AM" +"%Y-%m-%d %H:%M.%S"
#2009-12-05 00:00.37
date -d "Dec 5, 2009 12:00:37 AM 2 year ago" +"%Y-%m-%d %H:%M.%S"
#2007-12-05 00:00.37
date -d "+1 day" +"%Y%m%d"   #显示前一天的日期 
date -d "-1 day" +"%Y%m%d"   #显示后一天的日期 
date -d "-1 month" +"%Y%m%d" #显示上一月的日期 
date -d "+1 month" +"%Y%m%d" #显示下一月的日期 
date -d "-1 year" +"%Y%m%d"  #显示前一年的日期 
date -d "+1 year" +"%Y%m%d"  #显示下一年的日期
date -s "2012-05-23 01:01:01" # s
```

## eval

eval会对后面的cmdLine进行两遍扫描，如果第一遍扫描后，cmdLine是个普通命令，则执行此命令；

如果cmdLine中含有变量的间接引用，则保证间接引用的语义

```sh
eval [参数]
#参数说明：参数不限数目，彼此之间用分号分开。
eval ls /;ls / #连接多个命令
```

![image-20210816235409653](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210816235409653-1629129256-eff1b0.png)

扫描2次

![image-20210816235802041](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210816235802041-1629129482-ca2eed.png)

![image-20210817000026256](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20210817000026494-1629129830-848ab3.png)

