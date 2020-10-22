# Dictionary 字典

## 创建字典

```python
# key不能重复 key有hash的过程

# 字典创建方式
var1 = {'king':100,"tom":90,"jack":80}
var2 = dict(king=100,tom=90)
print(var1)
print(var2)
print(type(var2))
# 创建空字典
var3 = {}
print(type(var3))
# {'king': 100, 'tom': 90, 'jack': 80}
# {'king': 100, 'tom': 90}
# <class 'dict'>
# <class 'dict'>

# 字典生成式
item = ["zzz","xxx","vvv"]
values = [11.1,22.2,33.3]
# zip 以短的list的长度为基准
gen = {item:value for item,value in zip(item,values)}
print(gen)
# {'zzz': 11.1, 'xxx': 22.2, 'vvv': 33.3}

# str.upper() 转大写
gen2 = {item.upper():value for item,value in zip(item,values)}
print(gen2)
# {'ZZZ': 11.1, 'XXX': 22.2, 'VVV': 33.3}
```

## 通过key获取value

```python
# 获取字典的值通过键
# key不存在报错 keyError
print(var1["king"])
# key不存在返回None
print(var1.get("tom"))
# key不存在返回默认值
print(var1.get("kkk",99))
# 100
# 90
# 99
```

## 判断键是否存在

```python
# 键在字典中是否存在
print("king" in var1)
print("king" not in var1)
# True
# False
```

## 删除新增修改

```python
# 删除指定的键
del var1["jack"]
print(var1)
# {'king': 100, 'tom': 90}


# 清空dict
var2.clear()
# 新增
var2['king']=100
print(var2)
# {'king': 100}

# 修改
var2['king'] = 90
print(var2)
var2.update({"king":101})
print(var2)
var2["cc"]=1
# 修改已有的 新增没有的
var2.update({"king":101,"cc":2,"vvvv":8})
print(var2)
# {'king': 90}
# {'king': 101}
# {'king': 101, 'cc': 2, 'vvvv': 8}
```

## 视图

```python
# 视图
print(var2.keys())
print(var2.values())
print(var2.items())

print(list(var2.keys()))
print(list(var2.items()))
# dict_keys(['king', 'cc', 'vvvv'])
# dict_values([101, 2, 8])
# dict_items([('king', 101), ('cc', 2), ('vvvv', 8)])

# ['king', 'cc', 'vvvv']
# [('king', 101), ('cc', 2), ('vvvv', 8)]
```

## 遍历

```python
# 遍历

for item in var2:
    # 获取键
    print(item,end=" ")
    # 获取值
    print(var2.get(item),end=" ")
print()
# king 101 cc 2 vvvv 8

keys = var2.keys()
for key in keys:
    print(key,end=" ")
print()
# king cc vvvv
```