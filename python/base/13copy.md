# Copy

## 浅拷贝

只拷贝外层对象 子对象不拷贝

## 深拷贝

子对象也拷贝

```python
#encoding UTF-8
import copy


class Cpu:
  def __init__(self ,name):
      self.name=name


class Computer:
    def __init__(self,cpu):
        self.cpu=cpu


cpu1=Cpu("cpu1")
cpu2=cpu1

print(id(cpu1))
print(id(cpu2))

computer = Computer(cpu1)
computer1 = copy.copy(computer)
print(id(computer),id(computer.cpu))
print(id(computer1),id(computer1.cpu))

computer2 = copy.deepcopy(computer)
print(id(computer),id(computer.cpu))
print(id(computer2),id(computer2.cpu))
# 1731688205520
# 1731688205520
# 1731688204560 1731688205520
# 1731688749376 1731688205520
# 1731688204560 1731688205520
# 1731688749952 1731689024144
```