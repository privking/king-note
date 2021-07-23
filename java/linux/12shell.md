# Shell

## shell解释器

```sh
# !/bin/bash（默认）
# !/bin/ksh
# !/bin/bsh
# !/bin/sh
```

## helloworld

```sh
#!/bin/bash
echo 'hello world'
```

## shell执行方法

- 方法1：`./shell.sh`
- 方法2：`sh shell.sh` 或者`bash shell.sh`
- 方法3：`source shell.sh`

## shell常见变量

- shell的变量直接使用，例如：`a=15` （**等号前后不能有空格**），调用变量的话 `$a` 或者 `${a}`
- `$?` 判断上一条命令执行的是否成功
  - 上一条命令成功返回0
  - 上一条命令失败返回1
- `$0` 返回脚本的文件名称
- `$1-$9` 返回对应的参数值
- `$*` 返回所有的参数值是什么
- `$#` 返回参数的个数

## 四则运算

```sh
  echo $[12 + 6]
  echo $((12 + 6))
  
  echo $[12 - 6]
  echo $((12 - 6)) 
  
  echo $[12 * 6]
  echo $((12 * 6))
  
  echo $((12 / 6))
  echo $[12 / 6]
  
  echo $((12 % 6))
  echo $[12 % 6]
```

## 条件判断

### 文件路径

- -e 目标是否存在（exist）
- -d 是否为路径（directory）
- -f 是否为文件（file）
- [ -e foer.sh ] || touch foer.sh #判断当前目录下是否有foer.sh这个文件，假如没有就创建出foer.sh文件
- **中括号两边都要空格**

### 权限

- -r 是否有读取权限（read）
- -w 是否有写入权限（write）
- -x 是否有执行权限（excute）
- [ -x 123.txt ] && echo '有执行权限'

## 整数比较

- -eq 等于（equal）
- -ne 不等于(not equal)
- -gt 大于（greater than）
- -lt 小于（lesser than）
- -ge 大于或者等于（greater or equal）
- -le 小于或者等于（lesser or equal）
- [ 9 -gt 8 ] && echo '大于'

## 字符串

- = 相等
- != 不相等
- [ 'kkkkk' != 'kkkk' ] && echo '不等于'

## example

```sh
#!/bin/bash
#比较两个数的大小

if [ $1 -ge $2 ]
then
        echo "$1 大于等于$2"
else
        echo "$1 小于$2"
fi        
```

## 单分支判断

```sh
if [ 条件判断 ];
    then
    执行动作
fi

if [ 条件判断 ];
    then
    执行动作
else
    执行动作
fi
```

## 多分支判断

```sh
if [条件判断];
    then
    执行动作
elif [条件判断];
    then
    执行动作
elif [条件判断];
    then
    执行动作
fi
```

## for循环控制

```sh
#for 变量名 in 值1 值2 值3
#do
#执行动作
#done

for i in 1 2 3 4
do 
echo "$i"
sleep 2
done

```

```sh
#for 变量名 in `命令`
#do
#执行动作	
#done
# seq 1 10  1~10	
for i in `seq 1 10`
do 
echo "$i"
sleep 2
done

for i in $(cat a.txt)
do 
ping -c 2 $i
done
#a.txt
#www.baidu.com
#www.taobao.com
#
```

```sh
#for ((条件))
#do
#执行动作
#done
#!/bin/bash
for (( i=1;i<11;i++))
do
echo "$i"
done
```

## case

```sh
case 变量 in 
值1 )
执行动作1
;;
值2 )
执行动作2
;;
值3 )
执行动作3
;;
* )
执行动作4
;;
esac
```

## while

```sh
#while [ 条件判断式 ]
#do
#    执行动作
#done
#!/bin/bash
i=0
sum=0
while [ $i -lt $1 ]
do
sum=$(($sum+$i))
i=$(($i+1))
done
echo "sum=$sum"
```

## 定义函数

不限制定义和调用顺序

```sh
function name() {
    statements
    [return value]
}
```

```sh
#!/bin/bash
function getsum(){
    local sum=0
    for n in $@
    do
         ((sum+=n))
    done
    return $sum
}
getsum 10 20 55 15  #调用函数并传递参数
echo $?   # --->100


#调用函数并传递参数，最后将结果赋值给一个变量
total=$(getsum 10 20 55 15)
echo $total
#也可以将变量省略
echo $(getsum 10 20 55 15)
```

## select...in

适合终端（Terminal）这样的交互场景

select 是无限循环（死循环），输入空值，或者输入的值无效，都不会结束循环，只有遇到 break 语句，或者按下 Ctrl+D 组合键才能结束循环

运行到 select 语句后，取值列表 value_list 中的内容会以菜单的形式显示出来，用户输入菜单编号，就表示选中了某个值，这个值就会赋给变量 variable，然后再执行循环体中的 statements（do 和 done 之间的部分）。

```sh
select variable in value_list
do
  statements
done
```

```sh
#!/bin/bash

echo "What is your favourite OS?"
select name in "Linux" "Windows" "Mac OS" "UNIX" "Android"
do
    case $name in
        "Linux")
            echo "Linux是一个类UNIX操作系统，它开源免费，运行在各种服务器设备和嵌入式设备。"
            break
            ;;
        "Windows")
            echo "Windows是微软开发的个人电脑操作系统，它是闭源收费的。"
            break
            ;;
        "Mac OS")
            echo "Mac OS是苹果公司基于UNIX开发的一款图形界面操作系统，只能运行与苹果提供的硬件之上。"
            break
            ;;
        "UNIX")
            echo "UNIX是操作系统的开山鼻祖，现在已经逐渐退出历史舞台，只应用在特殊场合。"
            break
            ;;
        "Android")
            echo "Android是由Google开发的手机操作系统，目前已经占据了70%的市场份额。"
            break
            ;;
        *)
            echo "输入错误，请重新输入"
    esac
done
```
