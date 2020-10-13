# xsync&&xcall

## xsync

xsync脚本基于rsync工具，rsync 远程同步工具，主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点，它只是拷贝文件不同的部分，因而减少了网络负担。

```
rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
常用参数：
-r, –recursive 对子目录以递归模式处理
-R, –relative 使用相对路径信息
-l, –links 保留软链结
-v, –verbose 详细模式输出，传输过程可见
```

在`/usr/local/bin` 下创建xsync文件

```sh
#!/bin/sh
# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
        echo no args...;
        exit;
fi

# 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
# 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir
# 获取当前用户名称
user=`whoami`
# 循环
for((host=1; host<=2; host++)); do
        echo $pdir/$fname $user@slave$host:$pdir
        echo ==================slave$host==================
        rsync -rvl $pdir/$fname $user@slave$host:$pdir
done
#Note:这里的slave对应自己主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。sh
```

最后chmod a+x xsync给文件添加执行权限即可

## xcall

该脚本用于在所有主机上同时执行相同的命令。

进入`/usr/local/bin`目录下，输入vim xcall，向里面添加：

```sh
#!/bin/sh
pcount=$#
if((pcount==0));then
        echo no args...;
        exit;
fi
echo ==================master==================
$@
for((host=1; host<=2; host++)); do
        echo ==================slave$host==================
        ssh slave$host $@
done
#Note:这里的master和slave都是对应自己主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。
```

最后chmod a+x xcall给文件添加执行权限即可。