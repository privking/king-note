# codeStructure

## if-else

```python
score = int(input("请输入分数：\n"))
# if-elif-else
if score > 100 or score < 0:
    print("error input")
elif score >= 90 and score <= 100:
    print("A")
elif 80 <= score < 90:
    print("B")
elif 70 <= score < 80:
    print("C")
else:
    print("D")
# 请输入分数：
# 78
# C
```

## 嵌套

```python
# 嵌套结构
vip = bool(input("是不是会员\n"))
prise = int(input("价格\n"))

if vip:
    if prise>100:
        print("9折")
    else:
        print("9.5折")
else:
    print("不打折")

# 是不是会员
# True
# 价格
# 200
# 9折
```

## 简化if-else

```python
# 简化if-else
num1=100
num2=200
print(str(num1)+"大于"+str(num2) if num1>num2 else str(num1)+"小于"+str(num2))
```

## 占位符

```python
# pass占位符 没有逻辑   但是不报错
if num1>num2:
    pass
else:
    print(111)
```