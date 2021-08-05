# VIM操作

## 显示行号

```
:set number
```



## 跳转到某一行

```
打开文件时直接光标指向10行
vim +10 test.txt
跳转到10行
:10
第一行
gg
最后一行
G
移动到行首 
0 或者g0
```



## 搜索

```
/something 搜索something
n：查找下一个
N:查找上一个
```



## 替换

```
:%s/old/new/g - 用new替换文件中所有的old
:s/old/new - 用new替换当前行第一个old。
:s/old/new/g - 用new替换当前行所有的old。
:n1,n2s/old/new/g - 用new替换文件n1行到n2行所有的old
```



## 删除

```
:m,nd剪切m行到n行的内容.
dG: 剪切光标以下的所有行。
```



## 分屏

```
:Sex filename  # 水平新开一个窗口
:Vex filename  # 垂直新开一个窗口

# 切换窗口
# 左下上右 h j k l
# 先按ctrl + W 再按 h j k l 切换窗口
```

## 光标移动

```
h j k l 左下上右
```

