# list

## list特点

-  列表元素按顺序有序排列
-  索引映射唯一一个数据
-  列表可以存储重复数据
-  任意数据类型混合存储
-  根据需要动态分配和回收内存



## list创建方式

```python
# 创建列表方式1 使用[]
lst1 = ['hello', 'world', 'haha', 95]
print(lst1)
# ['hello', 'world', 'haha', 95]


# 创建列表方式2 使用list()
lst2 = list(['hello', 'world', 66])
print(lst2)
# ['hello', 'world', 66]
```

## list获取元素

```python
# 从左往右 正数索引  0开始
# 从右往左 负数索引  -1开始
print(lst2[1], lst2[-1])
# world 66
```

## list获取指定元素下标

```python
# 有多个相同元素  返回第一个
print(lst2.index("hello"))
# 0

# 查找索引在0-2内不包括3中的"world" 下标
print(lst2.index("world", 0, 3))
# 1
```

## list获取列表中多个元素

```python
# 列表名[start:stop:step]

# 切片的结果是原列表的拷贝
# step默认为1
# step为正数
#   [:stop:step] 切片的第一个元素是列表的第一个元素
#   [start::step] 切片的最后一个元素是列表的最后一个元素
#   从start开始往后切
# step为负数
#   [:stop:step] 切片的第一个元素是列表最后一个元素
#   [start::step] 切片的最后一个元素是列表的第一个元素
#   从start往前切
print(lst2[1:2:1])
# ['world']
print(lst2[1:2])
# ['world']
print(lst2[1:2:])
# ['world']
```

## list中判断元素是否存在

```python
# 判断一个元素是不是在列表里面
print("hello" in lst2)
# True
print("hello" not in lst2)
# False
```

## list新增元素

```python
# 向列表尾部插入一个元素
# 不创建新的列表对象，在原有对象上新增
lst2.append("king")
print(lst2)
# ['hello', 'world', 66, 'king']

# 将lst2所有元素添加到lst1末尾
lst1.extend(lst2)
print(lst1)
# ['hello', 'world', 'haha', 95, 'hello', 'world', 66, 'king']

# 向列表的任意位置插入元素
lst1.insert(0, "private")
print(lst1)
# ['private', 'hello', 'world', 'haha', 95, 'hello', 'world', 66, 'king']

# 列表的剪切
# 将lst1的1后面的元素都剪切掉，然后将list2添加到lst1后
lst1[1:] = lst2
print(lst1)
# ['private', 'hello', 'world', 66, 'king']
```

## list删除元素

```python
# 删除元素
# remove 移除第一个元素
lst1.remove("private")
print(lst1)
# ['hello', 'world', 66, 'king']

# pop
# 删除指定索引上的元素
# 不指定索引就删除最后一个元素
# 指定索引不存在 抛出IndexError

lst1.pop(0)
print(lst1)
# ['world', 66, 'king']
lst1.pop()
print(lst1)
# ['world', 66]


# 切片方式删除 （会产生新的列表对象）
print(lst1[0:1])
# ['world']
# 切片删除但是不新产生对象
lst1[1:2] = []
print(lst1)
# ['world']

# 清除列表中所有元素
lst1.clear()

# 删除列表对象
del lst1
```

## list修改元素

```python
# 修改值
print(lst2)
lst2[1] = "new"
print(lst2)
# ['hello', 'new', 66, 'king']
lst2[1:2]=[1,2,3,4]
print(lst2)
# ['hello', 1, 2, 3, 4, 66, 'king']
```

## list排序

```python
lst3=[1,3,2,4,6,5]
# 列表排序
# 不产生新对象
lst3.sort()
print(lst3)
# [1, 2, 3, 4, 5, 6]
lst3.sort(reverse=True)
print(lst3)
# [6, 5, 4, 3, 2, 1]

# 排序产生新对象（内置函数）
lst4 = sorted(lst3)
print(lst4)
# [1, 2, 3, 4, 5, 6]
```

## list生成式

```python
# 列表生成式
gen = [i*2 for i in range(1,10)]
print(gen)
# [2, 4, 6, 8, 10, 12, 14, 16, 18]
```