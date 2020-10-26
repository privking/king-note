# function 函数

## 函数的创建

```python
def 函数名([输入参数]):
	函数体
    [return xxx] 
```

```python
def calc(a,b):
	c=a+b
	return c
```

## 函数参数传递

```python
def func1(a, b):
    return a - b

# 根据位置传参
var1 = func1(1, 2)
print(var1) # -1

# 根据关键字传参
print(func1(b=1, a=2)) # 1
```

## 参数传递类型

```python
def func2(arg1,arg2):
    print(arg1)
    print(arg2)
    arg2.append(1)
    arg1=1

var2= 11
var3=[11,22,33]
func2(var2,var3)
print(var2) # 基本类型传值(不可变类型) 没修改到
print(var3) # 传的地址(可变类型) 所以修改到了
# 11
# [11, 22, 33, 1]
```

## 函数返回多个值

```python
def func3(num):
    odd=[]
    even=[]
    for i in num:
        if i%2:
            odd.append(i)
        else :
            even.append(i)
    return odd,even
print(func3([1,2,3,4,5,6,8,9]))
# 当返回多个参数时 封装成元组
# ([1, 3, 5, 9], [2, 4, 6, 8]) 
```

## 函数参数默认值

```python
# 定义带默认值函数
def func4(a,b=10):
    return a+b


print(func4(10)) # b使用默认值
print(func4(10, 20))
# 20
# 30
```

## 可变位置形参

```python
# 可变位置形参 封装成元组 类似于 java中...
def func5(*args):
    print(args)

func5(1)
func5(1,2)
func5(1,2,3)
# (1,)
# (1, 2)
# (1, 2, 3)
```

## 可变位置关键字形参

```python
# 可变位置关键字形参 封装成字典
def func6(**args):
    print(args)

func6(a=1,b=2)
func6(kk=1,vv=1)
# {'a': 1, 'b': 2}
# {'kk': 1, 'vv': 1
```

## 多种可变类型形参组合

```python
# 可变形参一个类型在一个方法里面只能有一个
# 不能同时有两个 * 或 ** 存在
def func7(*args1,**args2):
    print(args1)
    print(args2)

func7(1,2,3,a=11,b=22)
# (1, 2, 3)
# {'a': 11, 'b': 22}
```

## 将list转换为位置实参传入

```python
func7(1,2,3,a=11,b=22)
# (1, 2, 3)
# {'a': 11, 'b': 22}

def func7(a,b,c):
    print(a,b,c)
    
lst=[1,2,3]
func7(*lst) 
# 在函数调用时，将列表中的每个元素转换为位置参数传入
# 如果不加* 就会认定为传入的lst为第一个参数a
```

## 将dict转为关键字实参传入

```python
# 将字典中的键值对转换为关键字实参传入
# key必须和func7中形参的名称匹配
dic2={'a':1,'c':2,'b':3}
# func7(a=1,b=2,c=3)
func7(**dic2)
```

## 指定特定的参数使用关键字实参传递

```python
def func8(a,b,*,c,d):
    print(a)
    print(b)
    print(c)
    print(d)

# c d必须使用关键字传参
func8(1,2,c=4,d=3)
```

## 变量作用域

```python
var4 = 1  # 全局变量
age = 1 
def fun9(a, b):
    global age  # 将age声明为全局变量 age必须在外部有定义
    c = a + b  # c:局部变量

    age = c
    print(c)
    print(var4)

print(age)
```

## 递归函数

```python
# 递归函数
def func10(n):
    if(n==1):
        return 2
    else:
        return n*func10(n-1)
print(func10(10))
```