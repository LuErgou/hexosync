---
title: Python3中的时间总结(time,datetime)
date: 2016-08-25 10:04:23
categories:
- Python
tags:
- Python
- time
---
{% cq %} Python中的时间格式真烦。 {% endcq %}

<!--more-->
python3.5
centos7

并不经常用到Python中的时间，偶尔用的时候经常需要google。
Python对于日期的处理主要是**time**和**datetime**两个模块

### 总结
> 先总结，好查阅。

**time**模块归类于Generic Operating System Services，基于Unix Timestamp。
**datetime**模块基于**time**进行了封装，提供更多函数。

这两个模块都处理事件，但是对于时间的封装却不同。
**time**模块提供**struct_time**类, 即（time tuple, p_tuple, 时间元组）
**datetime**模块常用**datetime**和**timedelta**类，也提供了**date**、**time**类

python中时间分为

|类型|所属|
|--------|-------|
|时间戳| |
|时间元组p_tuple|time模块|
|datetime对象|datetime模块（又包含date对象和time对象)|
|timedelta对象|datetime模块| 

**常用**

#### 生成time tuple类对象p_tuple以及格式化输出
```
>>> time.localtime()   # 参数为时间戳，默认为当前时间
<class 'time.struct_time'> time.struct_time(tm_year=2016, tm_mon=8, tm_mday=18, tm_hour=10, tm_min=48, tm_sec=55, tm_wday=3, tm_yday=231, tm_isdst=0)

# p_tuple to str, 参数是format, p_tuple(默认为now)
>>>time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())     
<class 'str'> 2016-08-18 10:57:11

# str to p_tuple, 参数是str，format
>>>str = '2016-08-18 10:57:11'
>>>time.strptime(str, "%a %b %d %H:%M:%S %Y")
<class 'time.struct_time'>...
```

#### 生成时间戳
```
>>>import time, datetime
>>>time.time()
>>>time.mktime()  # 参数是p_tuple  # 默认是当前时间
>>>datetime.datetime.timestamp()  # 参数是datetime对象，默认是当前时间
```

#### 生成datetime类对像以及格式化输出
```
>>>datetime.datetime.now([tz]) #参数是时区，如果提供，则返回指定时区的本地时间
>>>datetime.datetime.today()   #返回当前时间的datetime对象
>>>datetime.datetime.utcnow()  #返回一个当前utc时间的datetime对象
>>>datetime.datetime.fromtimestamp(timestamp[,tz]) #根据时间戳
<class 'datetime.datetime'>

# datetime to str，datetime对象. strftime (format)
>>>d = datetime.datetime.now()
>>>d.strftime("%Y-%m-%d %H:%M:%S")
'2015-01-12 23:13:08'

# str to datetime
>>>str = '2014-12-31 18:20:10'
>>>datetime.datetime.strptime(str,"%Y-%m-%d %H:%M:%S")
datetime.datetime(2014, 12, 31, 18, 20, 10)
```

#### datetime.timedelta类
时间差，**datetime**模块中datetime对象、date对象和time对象相加减的结果是timedelta对象
timedelta接受多种参数
class datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, minutes=0, hours=0, weeks=0)
```
>>>td = datetime.timedelta(hours=3, days=1, minutes=20)
>>>td
1 day, 3:20:00
>>>td.days
1
>>>td.seconds
12000
```

#### p_tuple 和 datetime对象的转换
```
# datetime to p_tuple
>>> import datetime
>>> datetime.datetime.now().timetuple()
time.struct_time(tm_year=2016, tm_mon=8, tm_mday=25, tm_hour=9, tm_min=30, tm_sec=49, tm_wday=3, tm_yday=238, tm_isdst=-1)

# p_tuple to datetime, 先换成时间戳
>>>ts = time.mktime(p_tuple)
>>>dt = datetime.datetime.fromtimestamp(ts)
```


### time
见[Python time模块详解](http://ludaming.com/posts/Python/python-module-time.html)

### datetime
To-do

### 日期格式化符号
| 符号 | 含义 |
|--------|-------|
|%y |两位数的年份表示（00-99）|
|%Y |四位数的年份表示（000-9999）|
|%m| 月份（01-12）|
|%d |月内中的一天（0-31）|
|%H| 24小时制小时数（0-23）|
|%I |12小时制小时数（01-12）|
|%M |分钟数（00=59）|
|%S |秒（00-59）|
|%a |本地简化星期名称|
|%A |本地完整星期名称|
|%b |本地简化的月份名称|
|%B| 本地完整的月份名称|
|%c| 本地相应的日期表示和时间表示|
|%j |年内的一天（001-366）|
|%p |本地A.M.或P.M.的等价符|
|%U |一年中的星期数（00-53）星期天为星期的开始|
|%w |星期（0-6），星期天为星期的开始|
|%W| 一年中的星期数（00-53）星期一为星期的开始|
|%x |本地相应的日期表示|
|%X| 本地相应的时间表示|
|%Z| 当前时区的名称|
|%%| %号本身|


本文链接: [http://ludaming.com/posts/Python/python-date-time.html](http://ludaming.com/posts/Python/python-date-time.html)
