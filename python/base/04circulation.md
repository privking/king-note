# circulation

## range迭代器

```python
# range
r1 = range(10)
print(r1)
# range(0, 10) 打印迭代器对象

print(list(r1))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
# 起始为0 步长为1

r2 = range(1, 6)
print(list(r2))
# [1, 2, 3, 4, 5]
# 指定起始和结束

r3 = range(1, 8, 2)
print(list(r3))
# [1, 3, 5, 7]
# 指定起始 结束 步长

# 判断在不在序列中 in ,not in
print(2 in r3)
# False
print(2 in list(r3))
# False

# range() 返回一个迭代器对象
# 不管有多长 占用空间都相同  因为只存储start stop step
```

## while

```python
a = 1
while a < 5:
    print(a, end=" ")
    a += 1
print("")
# 1 2 3 4

b = 0
i1 = 1
while i1 < 10:
    b += i1
    i1 += 1;
print(b)
# 45
```

## for-in

```python
for i2 in range(1, 10):
    print(i2, end=" ")

print("")
# 1 2 3 4 5 6 7 8 9
for i3 in "king":
    print(i3, end=" ")
print("")
# k i n g

# 如果在循环找不需要使用变量 可以使用 ‘_’ 替代
for _ in range(10):
    print("rb", end=" ")
print("")
# rb rb rb rb rb rb rb rb rb rb
```

## break&&continue

```python
# 流程控制语句 break
# 流程控制语句 continue

for i4 in range(10):
    if (i4 == 1):
        continue
    if (i4 == 3):
        break
    print(i4)
# 0
# 2

# while else
# for else
# 如果在循环中没有遇到break 就执行else
for i5 in range(2):
    print(i5,end=" ")
else:
    print("执行break")
# 0 1 执行break
```