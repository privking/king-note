# module

## 引入一个模块

```python
import math
# import math as m

print(math.ceil(9.1))

```

## 引入一个方法，变量，类

```python
from math import ceil

print(ceil(9.1))
```

## 引入自己的模块

**make directory as source root**

clac2.py

```
def add(a, b):
    return a + b
```

demo.py

```python
import calc2

print(calc2.add(1, 21))
```

## 以主程序方式运行

每个模块定义中都包括一个记录模块名称的变量`__name__`,程序可以检查该变量，以确定他们在哪个模块中执行，如果一个模块不是被导入到其他程序中执行，那么它可能在解释器的顶级模块中执行，顶级模块的`__name__`变量值为`__main__`

calc1.py

```python
def add(a, b):
    return a + b

print("111111111")

```

calc2.py

```python
def add(a, b):
    return a + b

if __name__ == '__main__':
    print(add(1, 1)) # 只有运行calc2时才执行
```

demo1.py

```python
import calc1

print(calc1.add(1, 21))
# 111111111
# 22
```

demo2.py

```
import calc2

print(calc1.add(1, 21))

# 22
```



## Python中的包

包时一个分层次的目录结构，它将一组功能相近的模块组织在一个目录下

作用1：代码规范，作用2：避免模块名冲突

包与目录的区别：包含`__init__.py`文件的目录称为包，不包含的称为目录

包的导入 `import 包名.模块名`

使用`import` 只能导入**包名或模块名**

使用`from...import `可以导入**包，模块，函数，变量**



## Python常用内置模块

![image-20201102000800790](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201102000800790-1604246888-5eb511.png)