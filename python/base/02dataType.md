# DataType

## 输出变量信息

```python
name = 'king'
print("value " + name)
print("标识(地址)", id(name))
print("类型", type(name))
# value king
# 标识(地址) 2727130101936
# 类型 <class 'str'>
```

## 常见数据类型

```python
# 常见数据类型
# int float bool str
var1 = 10
print(var1, type(var1))
# 二进制
var2 = 0b100
print(var2, type(var2))
# 八进制
var3 = 0o100
print(var3, type(var3))
# 十六进制
var4 = 0x100
print(var4, type(var4))
# float
var5 = 3.14
print(var5, type(var5))
# 10 <class 'int'>
# 4 <class 'int'>
# 64 <class 'int'>
# 256 <class 'int'>
# 3.14 <class 'float'>

from decimal import Decimal
# Decimal
print(Decimal('1.1') + Decimal('2.2'))
print(1.1 + 2.2)
# 3.3
# 3.3000000000000003

var6 = True
print(var6, type(var6))  # True <class 'bool'>
var7 = False
print(var7, type(var7))  # False <class 'bool'>
# 可以相加
print(var6 + 1)  # True->1
print(var7 + 1)  # False->0


str1 = 'hello world'
print(str1, type(str1))
str2 = "hello world"
print(str2, type(str2))
str3 = """hello
 world"""
print(str3, type(str3))
str4 = '''hello
 world'''
print(str4, type(str4))
# hello world <class 'str'>
# hello world <class 'str'>
# hello
#  world <class 'str'>
# hello
#  world <class 'str'>
```

## 类型转换

```python
var8 = "king"
var9 = 9
print(var8 + str(var9))
print(type(str(var9)))
print(int(True), type(int(True)))
print(float(var9), type(float(var9)))
# king9
# <class 'str'>
# 1 <class 'int'>
# 9.0 <class 'float'>
```

## 注释

```python
# 单行注释
'''
多行注释
多行注释
'''
# coding:UTF-8
# 在文件开头写 指定编码格斯
```

## 输入

```python
var10 = input("请输入一个数\n")
print(var10, type(var10))
# 请输入一个数
# 111
# 111 <class 'str'>
```

## 运算符

```python
print(1 + 1)  # 加法
print(1 - 1)  # 减法
print(2 * 4)  # 乘法
print(5 / 2)  # 除法

print(5 % 2)  # 取余
print(5 % -2)  # 负数取余 被除数-除数*商 商->整除
print(5 // 2)  # 整除 不管正负都向下取整
print(5 // -2)

print(5 ** 2)  # 乘方

# 2
# 0
# 8
# 2.5
# 1
# -1
# 2
# -3
# 25
```