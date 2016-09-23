---
title: python3 列表生成式，生成器generator和yield
date: 2016-09-23 14:09:55
categories:
- Python
tags:
- Python
- 编程
---


列表生成式，生成器，yield
<!--more-->

## 概述
python中我们一定会用到列表。
有些有规律的列表我们可以使用列表生成式来表示，列表生成式（List Comprehensions）是Python内置的非常简单却强大的可以用来创建list的生成式。
如果一个有规律的列表的长度过大，就可以使用生成器（generator）来表示，使用生成器不会直接生成整个列表，节省内存。
yield关键字可以定义生成器，一个函数中如果存在yield关键字，那这个函数就是一个生成器。

## 列表生成式
### 不使用列表生成式
```
# 简单列表
list1 = list(range(1,10)
# 复杂列表，例如x的平方
list2 = []
for x in range(1,10):
    list2.append(x*x)
```
### 使用列表生成式
```
# 等同于上面那个循环
list3 = [x * x for x in range(1,10)]

# for循环后还可以加入判断，例如只要奇数平方
list4 = [x * x for x in range(1,10) if x %2 != 0]

# 使用两层循环
list5 = [x + y for x in 'AB' for y in [abc]]
-->['Aa','Ab','Ac',...]

```

使用列表生成式可以让代码更简洁。


## 生成器generator
列表生成式是创建出一个列表。
由于内存限制，一个列表的长度是有限的。
如果我们需要一个列表，它的长度很长然而我们并不会访问所有元素。
仍然使用列表生成器就会造成空间的浪费

使用生成器，可以不用创建完整的列表，而是在循环的过程中（列表生成式是循环）算出我们所需要位置的值

### 可以由列表生成式表达的列表，[] ->()
将列表生成式的[] 换为 ()即可

```
list6 = (x*x for x in range(1,10))

# 直接访问
list6
--><generator object <genexpr> at 0x1022ef630>

# 使用next访问，当计算到最后一个元素后继续调用next时会抛出StopIteration错误（next方法基本不会用到）
next(list6)
-->1
next(list6)
-->4
...
...
# 使用for循环访问
for i in list6:
    print(i)

```

### 复杂的算法（无法用列表生成式实现）时，使用函数来实现
例如斐波拉契数列（Fibonacci），除了前两个数，任意一个数都可由前两个数相加得到：
1, 1, 2, 3, 5, 8, 13, 21, 34, ...
无法使用for循环写出来。只能用函数

```
# 打印出数列中前num个数字的函数
def fib1(num):
    n, a, b = 0, 0, 1
    while n <last:
        print(b)
        a, b = b, a+b
        n += 1
    return 0

```

将函数变为生成器很简单，只要函数中包含yield关键字，这个函数就是一个生成器，会返回yield关键字声明的变量

```
def fib2(num):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

理解生成器和函数的执行流程
- 函数：
1. 调用函数。
2. 顺序执行代码，遇到return语句就返回，或者执行到代码末尾结束。
3. 返回结果
- 生成器：
1. 生成一个generator对象
2. 用next()函数、for循环获得值（next方法基本不会用到）
3. 返回generator对象，在遍历该对像时，计算元素。

生成器对象在执行过程中，遇到yield关键字就会中断并返回yield定义的变量，下次再调用，就会从上次中断的地方继续执行，直到下一个yield关键字。

## 小结
它是在for循环的过程中不断计算出下一个元素，并在适当的条件结束for循环。对于函数改成的generator来说，遇到return语句或者执行到函数体最后一行语句，就是结束generator的指令，for循环随之结束。



本文链接[http://ludaming.com/posts/Python/python-generator.html](http://ludaming.com/posts/Python/python-generator.html)
我的博客: [ludaming.com](http://ludaming.com)



