

# print

## print()方法

```python
# 输出字符串
print("hello word")
# 输出数字
print(123)
# 输出表达式
print(123 + 123)
print("fff" + "vvv")
# 完整输出
# sep 分割符
# file 指定输出
# end 指定结尾
# flush 动态刷新
print("aa", "bbb", "cc", sep="_", file=sys.stdout, end="vvv\n", flush=False)
# 默认以空格为分割符
# print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
print("1", "2", "3")
# 不换行
print('1111', end='')
print('222')

# hello word
# 123
# 246
# fffvvv
# aa_bbb_ccvvv
# 1 2 3
# 1111222
```

## 格式化输出

### %格式化

```python
print('hello %s' % 'king')
print('h%s %s' % ('hello', 'world'))
str1 = "the length of (%s) is %d" % ('king', len('king'))
print(str1)

# hello king
# hhello world
# the length of (king) is 4
```



| 符  号 | 描述                                 |
| :----- | :----------------------------------- |
| %c     | 格式化字符及其ASCII码                |
| %s     | 格式化字符串                         |
| %d     | 格式化整数                           |
| %u     | 格式化无符号整型                     |
| %o     | 格式化无符号八进制数                 |
| %x     | 格式化无符号十六进制数               |
| %X     | 格式化无符号十六进制数（大写）       |
| %f     | 格式化浮点数字，可指定小数点后的精度 |
| %e     | 用科学计数法格式化浮点数             |
| %E     | 作用同%e，用科学计数法格式化浮点数   |
| %g     | %f和%e的简写                         |
| %G     | %f 和 %E 的简写                      |
| %p     | 用十六进制数格式化变量的地址         |

### format格式化

#### 占位符

```python
# 格式化输出 format
print('{} {}'.format('WW', 'QQ'))
# 可打乱顺序
print('{1} {0}'.format('AA', 'SS'))
# 可重复
print('{1} {0} {1}'.format('AA', 'SS'))
# 带关键字
print('{a} {b}'.format(a='bb', b='pp'))
# 可在字符串前加f以达到格式化的目的，在{}里加入对象
salary = 9999.99
print(f'My salary is {salary:10.3f}')

# WW QQ
# SS AA
# SS AA SS
# bb pp
# My salary is   9999.990
```



#### 格式转换

```python
# 格式转换
print('{0:b}'.format(10))  # 'b' - 二进制。将数字以2为基数进行输出。
# 1010
```

1. 'b' - 二进制。将数字以2为基数进行输出。
2. 'c' - 字符。在打印之前将整数转换成对应的Unicode字符串。
3. 'd' - 十进制整数。将数字以10为基数进行输出。
4. 'o' - 八进制。将数字以8为基数进行输出。
5. 'x' - 十六进制。将数字以16为基数进行输出，9以上的位数用小写字母。
6. 'e' - 幂符号。用科学计数法打印数字。用'e'表示幂。
7. 'g' - 一般格式。将数值以fixed-point格式输出。当数值特别大的时候，用幂形式打印。
8. 'n' - 数字。当值为整数时和'd'相同，值为浮点数时和'g'相同。不同的是它会根据区域设置插入数字分隔符。
9. '%' - 百分数。将数值乘以100然后以fixed-point('f')格式打印，值后面会有一个百分号。



#### 格式

```python
print('{} is {:.2f}'.format(1.123, 1.123))  # 取2位小数
print('{:<30}'.format('left aligned'))  # 左对齐（默认）
print('{:>30}'.format('right aligned'))  # 右对齐
print('{:^30}'.format('centered'))  # 中间对齐
print('{:*^30}'.format('centered'))  # 使用“*”填充
print('{:0=30}'.format(11))  # 还有“=”只能应用于数字，这种方法可用“>”代替
print('{:+f}; {:+f}'.format(3.14, -3.14))  # 总是显示符号
print('Correct answers: {:.2%}'.format(7 / 8))  # 打印百分号

# 格式化时间
# import datetime
d = datetime.datetime(2010, 7, 4, 12, 15, 58)
print('{:%Y-%m-%d %H:%M:%S}'.format(d))

# 1.123 is 1.12
# left aligned
#                  right aligned
#            centered
# ***********centered***********
# 000000000000000000000000000011
# +3.140000; -3.140000
# Correct answers: 87.50%
# 2010-07-04 12:15:58

```

