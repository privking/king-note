# extends

```python
class 子类名称（父类1,父类2...）:
	pass
```

- 如果一个类没有继承任何类,默认继承object
- Python支持多继承
- 定义子类时必须调用父类的构造函数

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def get_name(self):
        return self.name

    def get_age(self):
        return self.age

    def info(self):
        print("person")


class Work:
    def __init__(self, work):
        self.work = work

    def get_work(self):
        return self.work

    def info(self):
        print("work")

# 注意子类如果没有构造方法时，按括号内父类的继承顺序去继承父类构造方法，只继承一个
# 按顺序继承，哪个父类在最前面且它又有自己的构造函数，就继承它的构造函数
# 如果最前面第一个父类没有构造函数，则继承第2个的构造函数，第2个没有的话，再往后找，以此类推。


# 需要注意圆括号中继承父类的顺序，若是父类中有相同的方法名，而在子类使用时未指定，python从左至右搜索
# 即方法在子类中未找到时，从左到右查找父类中是否包含方法
# python函数没有重载

# super().__init__相对于类名.__init__，在单继承上用法基本无差

class Desc(Person, Work):
    def __init__(self, name, age, work):
        # super().__init__(name=name,age=age)
        Person.__init__(self,name, age)
        Work.__init__(self,work)

    # 方法重写
    def info(self):
        print("desc")

desc = Desc("king", 11, "programmer")
print(desc.get_name())
print(desc.get_work())
print(desc.get_age())
desc.info()
# king
# programmer
# 11
# desc
```