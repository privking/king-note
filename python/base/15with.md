# With

```python
class ContextMgr :
    # 实现了 __enter__ 和 __exit__称为遵循了上下文管理器
    def __enter__(self):
        print("调用enter 方法")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("调用exit方法")
        print(exc_type)
        print(exc_val)
        print(exc_tb)

    def myfunc(self):
        print("调用myfunc")
        return "aaaaa"


mgr = ContextMgr()
mgr.myfunc()

with ContextMgr() as mgr2:
    print(mgr2.myfunc())
   
    
# 调用myfunc
# 调用enter 方法
# 调用myfunc
# aaaaa
# 调用exit方法
# None
# None
# None
```

```python
class ContextMgr :
    # 实现了 __enter__ 和 __exit__称为遵循了上下文管理器
    def __enter__(self):
        print("调用enter 方法")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("调用exit方法")
        print(exc_type)
        print(exc_val)
        print(exc_tb)

    def myfunc(self):
        print("调用myfunc")
        return "aaaaa"


mgr = ContextMgr()
mgr.myfunc()

with ContextMgr() as mgr2:
    print(mgr2.myfunc())
    raise Exception("exc")
    
    
# 调用myfunc
# 调用enter 方法
# 调用myfunc
# aaaaa
# 调用exit方法
# <class 'Exception'>
# exc
# <traceback object at 0x00000251ED91C700>
# Traceback (most recent call last):
#   File "D:/python/study/priv/king/with/MyContextMgr.py", line 25, in <module>
#     raise Exception("exc")
# Exception: exc
```

