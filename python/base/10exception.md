# exception

```python
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
finally:
<语句>        #最终会执行
```

## simple

```python
try:
    print(1/0)
except ZeroDivisionError:
    print("不能除0")
```

## 多except

```python
try:
    print(1 / 0)
except ZeroDivisionError:
    print(11111)
except ValueError:
    print(222222)
```

### 不带任何异常类型

```python
# 捕获所有发生的异常
try:
    print(1/0)
except:
    print("不能除0")
```

## 带else

```python
# 没有发送异常就会执行 else
try:
    pass
except ZeroDivisionError:
    print(1)
else:
    print("else")
```

### 带finally

```python
# 不管有没有异常都会执行finally
try:
    pass
except ZeroDivisionError:
    print(1)
else:
    print("else")
finally:
    print("final")
```

### 带参数

```python
try:
    print(1 / 0)
except ZeroDivisionError as e:
    print(e)
    
    
# 在python2中用 ,
try:
    print(1 / 0)
except ZeroDivisionError , e:
    print(e)
```

###  注意点

![image-20201028145723941](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201028145723941-1603868270-508bb4.png)

![image-20201028145731366](C:\Users\58443\AppData\Roaming\Typora\typora-user-images\image-20201028145731366.png)

### 抛出异常

```python
def functionName( level ):
    if level < 1:
        raise Exception("Invalid level!", level)
        # 触发异常后，后面的代码就不会再执行
```

### 自定义异常

```python
class Networkerror(RuntimeError):
    def __init__(self, arg):
        self.args = arg
```

