# Class

![image-20201028174017978](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201028174017978-1603878018-3617a2.png)

## 创建一个类

```python
class Student:
    """
    类注释
    """
    # 类的变量
    class_param="类的变量"
    # protect 变量
    # 不能用于 from module import *
    _protect_param="protect"
    # private 变量
    __private_param="private"

    def __init__(self,name,age):
        # 对象的变量
        self.name=name
        self.age=age

    # 定义普通方法
    def info(self):
        print(f"name: {self.name}, age:{self.age}")

    # 定义类的方法
    @classmethod
    def class_method(cls):
        print(cls.class_param)

    # 定义静态方法
    @staticmethod
    def static_method():
        print("staticMethod")

    # 运算符重载
    # 重载 +
    def __add__(self, other):
        var=Student("",0)
        var.age=self.age+other.age
        var.name=self.name+other.name
        return var

    # toString()
    def __str__(self):
        return f"age:{self.age},name:{self.age}......."

    # private 私有方法
    def __hello(self):
        print("hello")

    # protect 方法
    def _hh(self):
        print("hh")

```

## new 对象

```python
stu = Student("king", 123)
```

```python
# encoding=UTF-8

# __new__ && __init__ 先后顺序

class Person(object):
    def __new__(cls, *args, **kwargs):
        print(f"new 方法调用 classId->", id(cls))
        obj = super().__new__(cls)
        print(f"创建对象的id->", id(obj))
        return obj

    def __init__(self):
        print(f"init方法调用 selfId->", id(self))


print(f"object的id->", id(object))
print(f"Person的id->", id(Person))
p = Person()
print(f"p的id为->", id(p))

# object的id-> 140708853345104
# Person的id-> 2377667763632
# new 方法调用 classId-> 2377667763632
# 创建对象的id-> 2377699643936
# init方法调用 selfId-> 2377699643936
# p的id为-> 2377699643936
```



## 方法调用

```python
stu = Student("king", 123)

# 对象调用普通方法
stu.info()
print("--------------")
# 类调用普通方法 需要自行传入self
Student.info(stu)

# 对象调用静态方法
stu.static_method()
# 对象调用类方法
stu.class_method()
# 类调用静态方法
Student.static_method()
# 类调用类方法
Student.class_method()
# 对象用 . 调用成员变量
print(f"name->{stu.name},age->{stu.age},classParam->{Student.class_param}")

# 对象和类都有自己的id 和type
print(id(stu))
print(type(stu))
print(id(Student))
print(type(Student))
# 2833866755280
# <class 'priv.king.object.Student.Student'>
# 2833836009840
# <class 'type'>

print("-----------")
# 实现了运算符重载
stu2 = Student("123", 123)
stu3=stu + stu2
print(stu3)
# age:246,name:246.......

print("---------------")
#内置属性
print(Student.__doc__) # 文档
print(Student.__name__) # 名称
print(Student.__module__) 
print(Student.__bases__) # 父类们
print(Student.__base__) # 靠的最近的父类
print(Student.__dict__) # 属性些 包括方法属性
print(stu.__dict__) # 变量属性
print(Student.__mro__) #继承结构
print(Student.__subclasses__) #子类
print("==============")
#访问私有变量
#Python不允许实例化的类访问私有数据，但你可以使用 object._className__attrName（ 对象名._类名__私有属性名 ）访问属性
print(stu._Student__private_param)
print(stu._protect_param)
# 访问私有方法
stu._Student__hello()
stu._hh()
```

## 动态绑定

```python
stu = Student("king", 123)

# 动态绑定属性（给对象）
stu.gender = "man"
print(stu.gender)

# 动态绑定属性（给类）
Student.zbc="ccc"
print(Student.zbc+"__")


# 动态绑定类方法
@classmethod
def func1(cls):
    print("这是类方法")

Student.func1=func1
Student.func1()
stu.func1()
# 动态绑定静态方法

@staticmethod
def func2():
    print("这是静态方法")

Student.func2=func2
Student.func2()
stu.func2()

# 普通方法
def func3(self):
    print("this %s's content is ............."%self.name)
stu.func3 = types.MethodType(func3,stu) #里面的参数是方法名，对象名
stu.func3()

a=types.MethodType(func3,stu)
a()

stu.a=types.MethodType(func3,stu)
stu.a()

# 运行的过程中删除属性、方法
# delattr(对象, "属性名") 注意属性名用引号引起
# del 对象.属性名
# del 和delattr功能有限，都是针对实例对象而言的，对于类方法，类属性则删除不了。
# 因为del和delattr两个方法主要用来删除绑定的实例属性和实例方法。
del stu.a
delattr(stu,"func3")


# man
# ccc__
# 这是类方法
# 这是类方法
# 这是静态方法
# 这是静态方法
# this king's content is .............
# this king's content is .............
# this king's content is .............
```

