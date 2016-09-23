---
title: python3 open()方法和文件读写
date: 2016-09-22 10:03:26
categories:
- Python
tags:
- Python
- 编程
- 文件处理
---
python3.5
centos7
关于python对于文件的读写和open()方法。
<!-- more -->


## open()方法
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
打开一个文件并返回文件对象
如果该文件无法被打开，会抛出OSError
官方[python document](https://docs.python.org/3/library/functions.html?highlight=open#open)

### 参数
- file: 必需，文件路径（相对或者绝对路径）。
- mode: 可选，文件打开模式
- buffering: 设置缓冲
- encoding: 一般使用utf8
- errors: 报错级别
- newline: 区分换行符
- closefd: 传入的file参数类型
- opener:

一般只用到file，mode和encoding


### mode参数
|方式|解释|
|----|----|
|'r'     |读模式（默认）|
|'w'   | 写模式，打开时会清空文件|
|'x'   |  写模式，新建一个文件，如果该文件已存在则会报错。|
|'a'  |   添加模式，写文件只能写到文件末尾，不能读|
|'b' |    二进制模式|
|'t' |    文本模式 (默认)|
|'+' |    打开一个文件进行更新(可读可写)|
|'U'|     通用换行模式（不推荐）|

- w:可读写，打开时清空文件
- r+:可读写，打开时不清空文件，可写到文件任何位置。默认在文件开始，因此会覆写文件
- a+:可读写，打开时不清空文件，只能写到文件末尾

默认为文本模式，如果要以二进制模式打开，加上'b'

### 注意
使用open()方法一定要保证关闭文件对象，即调用close()方法
>当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用close()方法时，操作系统才保证把没有写入的数据全部写入磁盘同时释放资源。忘记调用close()的后果是数据可能只写了一部分到磁盘，剩下的丢失了。


## 使用with ... as ...
正常情况下，想要打开一个文件并且保证该文件会被关闭。我们需要
```
try:
    f = open('/path/to/file', 'r')
    # do something about f
finally:
    if f:
        f.close()
```
使用with...as...能确保文件一定被关闭。
```
with open('/path/to/file', 'r') as f:
    f.read()
    ...
```


## 文件读写

### 读文件
```
with open('path/to/file', 'r', encoding='utf8) as f:
    f.read()
    f.readline()
    f.readlines()
```
- read([size]): 读出指定大小的内容，默认为读取所有。（小心内存爆炸）
- readline(): 读出一行。
- readlines(): 读出所有，返回值是是一个list。


### 写文件
```
with open('path/to/file', 'r', encoding='utf8) as f:    
   for item in sql_list:
       f.write(item+';\n')
```
- write(): 写入文件，可以是字符串。


## 小结
python通过open()函数打开的文件对象进行文件操作
打开文件的时候注意打开的模式
使用with...as...是推荐的


本文链接[http://ludaming.com/posts/Python/python-open-file.html](http://ludaming.com/posts/Python/python-open-file.html)
我的博客: [ludaming.com](http://ludaming.com)
