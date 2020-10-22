# Tuple 元组

```python
# 元组 不可变序列
# 在多任务环境下，同时操作对象不需要加锁
# 如果元组中对象本身是不可变对象，则不能再引用其他对象
# 如果元组中的对象是可变对象（比如列表），则可变对象的引用不允许改变，但是数据可以改变

# 创建元组
var1 = ("king","tom",1)
print(var1)
print(type(var1))

var2 = tuple(("hello","world",11))
print(var2)
print(type(var2))

# 只有一个元素时。必须加逗号,不然就不是元组是原始类型
var3 = ("king",)
print(type(var3))

# 空元组
var4 = ()
print(type(var4))
var5 = tuple()
print(type(var5))

# ('king', 'tom', 1)
# <class 'tuple'>
# ('hello', 'world', 11)
# <class 'tuple'>
# <class 'tuple'>
# <class 'tuple'>
# <class 'tuple'>

# 遍历
for item in var1:
    print(item,end=" ")
print()
# king tom 1 
```

