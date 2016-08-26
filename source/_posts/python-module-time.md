---
title: Python time模块详解
date: 2016-08-25 16:46:44
categories:
- Python
tags:
- Python
- Python模块
- time
- 编程
- 时间
---
{% cq %} Python的time模块详解。 {% endcq %}

<!--more-->
官方手册对于**time**模块的说明:
>尽管此模块大部分时候是有效的，但是有的函数在某些平台上无效。（说的就是你，Windows）
此模块中大部分方法调用的名称本地平台下的C语言库同名


## 概念
- 时间戳
1.时间戳以自从1970年1月1日午夜（历元）经过了多长时间来表示。
2.浮点数
3.值得注意的是时间戳只能表示1970-2038年之间的时间。
- 时间元祖(p_tuple)
**time**模块自带的struct_time类对象，是由9组数字组成的元组
|属性|字段|值|
|----|----|----|
|tm_year| 4位数年 |  2008|
|tm_mon|  月|  1 到 12|
|tm_mday|   日| 1 到 31|
|tm_hour  | 小时| 0 到 23|
|tm_min   | 分钟|0 到 59|
|tm_sec   | 秒|0 到 61 (61 是闰秒)|
|tm_wday|  一周的第几日|  0到6 (0是周一)|
|tm_yday  |  一年的第几日|1 到 366(儒略历)|
|tm_isdst |   是否为夏令时|-1, 0, 1|
>今年的最后一天会多出一个闰秒来着

## 获取时间戳
```
>>>import time
>>>time.time()   # 不接受参数，当前时间的时间戳
1472112974.4033062    
>>>time.mktime(p_tuple)  # 接受时间元组，返回时间戳
1472112974.4033062  
```

## 获取时间元组
```
>>>time.localtime([secs])  # 返回给定时间戳对应的本地时间下的时间元祖，默认为当前时间
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=25, tm_hour=16, tm_min=17, tm_sec=4, tm_wday=3, tm_yday=238, tm_isdst=0)
>>>time.gmtime([secs])  # 返回给定时间戳对应的UTC时间下的时间元祖，默认为当前时间
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=25, tm_hour=8, tm_min=24, tm_sec=41, tm_wday=3, tm_yday=238, tm_isdst=0)
```

## 获取时间字符串
```
>>>time.asctime([p_tuple])  # 返回给定时间的字符串，默认为当前时间
'Thu Aug 25 16:26:53 2016'
>>>time.ctime([secs])         # 同asctime，接受参数为时间戳
'Thu Aug 25 16:26:53 2016'
```

## 时间格式化
time.strftime(format[, p_tuple])  #参数为格式化符号，时间元祖。默认为本地当前时间，返回字符串
time.strptime(str,format) # 将字符串转为时间元组
```
>>>time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
'2016-08-25 16:30:58'
>>>s = '2016-08-25 16:30:58'
>>>time.strptime(s, "%Y-%m-%d %H:%M:%S")
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=25, tm_hour=16, tm_min=34, tm_sec=12, tm_wday=3, tm_yday=238, tm_isdst=-1)
```

## time.clock
time.clock()返回当前cpu时间，常用于计算一个程序的耗时
如果要计算一段代码的执行时间，我们可以
```
t1 = time.time()
blabla代码
t2 = time.time()
t2 - t1
```
但是如果执行的代码中有time.sleep()之类阻塞线程的代码
我们就无法知道该代码准确的cpu时间，这时可以用time.cllock()
```
t1 = time.clock()
time.sleep(5)
blabla代码
t2 = time.clock()
t2 - t1
```
这样就会忽视掉阻塞的时间，输出cpu计算的时间
>但是，在windows下，time.clock()返回的和time.time()一样，都是真实时间(wall-time)
所以windows真的很烦


## 其他
|函数/属性|说明|
|----|----|
|time.sleep(secs)|阻塞调用线程的进行|
|time.clock()|返回当前cpu时间，常用于计算一个程序的耗时|
|time.tzset()|根据环境变量，初始化时间设置|
|time.altzone|格林威治西部的夏令时地区的偏移秒数|
|time.timezone|当地时区（未启动夏令时）距离格林威治的偏移秒数 |
|time.tzname|属性time.tzname包含一对根据情况的不同而不同的字符串，分别是带夏令时的本地时区名称，和不带的。|

## 日期格式化符号
在这里有，不重复写了
[Python中的时间总结(time,datetime)](http://ludaming.com/posts/Python/python-date-time.html)


本文链接: [http://ludaming.com/posts/Python/python-module-time.html](http://ludaming.com/posts/Python/python-module-time.html)
