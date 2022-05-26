# Set 集合

```python
# 集合是可变序列
# 没有value 的字典
# 不允许重复

# 创建set
var1 = {"king","tom","jack"}
print(var1)

var2 = set(range(6))
print(var2)

var3 = set([1,2,3,4])
print(var3)

var4 = set((1,2,3,3))
print(var4)

var5 = set('king')
print(var5)

# 定义空集合
var6 = set()
# {'tom', 'jack', 'king'}
# {0, 1, 2, 3, 4, 5}
# {1, 2, 3, 4}
# {1, 2, 3}
# {'g', 'i', 'n', 'k'}


# 判断是否在集合中
print("king" in var1)
# True


# 添加元素
var1.add("bbb")
var1.update([1,2,3])
var1.update({'oo','pp'})
print(var1)
# {1, 2, 3, 'king', 'jack', 'tom', 'oo', 'bbb', 'pp'}

# 删除
var1.discard('oo')
var1.pop()
print(var1)
# {1, 2, 3, 'tom', 'king', 'pp', 'jack'}


var7={'11','22'}
var8={'11','22'}
print(var7==var8)
# True 元素相同就相等
# var7是否是var8的子集

print(var7.issubset(var8))
# var7是否是var8的超集
print(var7.issuperset(var8))
# var7与var8是否有交集
print(var7.isdisjoint(var8))
# True
# True
# False

# 求交集
var7.intersection(var8)
var9 = var7 & var8

# 求并集
var7.union(var9)
var10 = var7 | var8

# 差集 （减法）
var7.difference(var8)
var11 = var7 - var8

# 对称差集
var7.symmetric_difference(var8)
var12 = var7 ^ var8


# 集合生成式
var13 = {i**i for i in range(6)}
print(var13)
# {256, 1, 4, 3125, 27}
```

