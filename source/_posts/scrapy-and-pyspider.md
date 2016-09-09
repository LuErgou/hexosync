---
title: scrapy和pyspider简介
date: 2016-09-06 16:16:15
categories:
- 爬虫
tags:
- spider
- 爬虫
- Python模块
- pyspider
- scrapy
---
{% cq %}人生苦短，少用windows{% endcq %}
<!-- more -->
一开始工作上要写爬虫
当时只听说过scrapy框架，据吹那是好用的一塌糊涂。
但不能你让我用scrapy我就用scrapy
我要自己试一下。
不支持python3。。。

requests+bs4+re就自己开写了，
然后用了多线程，又改成多进程。
总感觉自己写的有点靠不住。

正好发现scrapy开始支持python3了。（然而只能在linux平台，因为scrapy依赖于twisted，而twisted在win平台不支持python3）
又听说了有国人搞出来的pyspider

应该比我自己写的不知道高到哪里去了。

所以还是学习一下框架比较好
pyspider和scrapy都试试。


查了一下，两者对比的评价
>pyspider的优点是简单，立刻就能上手，脚本编写规则。懂了的话，一小时写甚至可以写十多个爬虫。
scrapy的优点是自定义程度高，适合学习研究爬虫技术，要学习的相关知识也较多，故而完成一个爬虫的时间较长。



## pyspider简介

- 通过python脚本进行结构化信息的提取，follow链接调度抓取控制，实现最大的灵活性
- 通过web化的脚本编写、调试环境。web展现调度状态
- 抓取环模型成熟稳定，模块间相互独立，通过消息队列连接，从单进程到多机分布式灵活拓展



pyspider架构图
![pyspider-arch](http://oc1suuvql.bkt.clouddn.com/image/jpgpyspider-arch.jpg)
 
|模块|功能|
|----|----|
|webui|web的可视化任务监控，web脚本编写，单步调试，异常捕获，log捕获，print捕获等|
|scheduler|任务优先级，周期定时任务，流量控制，基于时间周期 或 前链标签（例如更新时间）的重抓取调度|
|fetcher|dataurl支持，用于假抓取模拟传递，method, header, cookie, proxy, etag, last_modified, timeout 等等抓取调度控制，可以通过适配类似 phantomjs 的webkit引擎支持渲染|
|processor|内置的pyquery，以jQuery解析页面，在脚本中完全控制调度抓取的各项参数，，可以向后链传递信息，异常捕获|
 
 
1.  各个组件间使用消息队列连接，除了scheduler是单点的，fetcher 和 processor 都是可以多实例分布式部署的。 scheduler 负责整体的调度控制
2.  任务由 scheduler 发起调度，fetcher 抓取网页内容， processor 执行预先编写的python脚本，输出结果或产生新的提链任务（发往 scheduler），形成闭环。
3.  每个脚本可以灵活使用各种python库对页面进行解析，使用框架API控制下一步抓取动作，通过设置回调控制解析动作。




## scrapy介绍
Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。
其最初是为了页面抓取 (更确切来说, 网络抓取 )所设计的， 也可以应用在获取API所返回的数据(例如 Amazon Associates Web Services ) 或者通用的网络爬虫。
Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试

scrapy架构图：
![scrapy-arch](http://oc1suuvql.bkt.clouddn.com/image/jpgscrapy-arch.jpg)

|组件|功能|
|----|----|
|引擎(Scrapy)|用来处理整个系统的数据流处理, 触发事务(框架核心)|
|调度器(Scheduler)|用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回. 可以想像成一个URL（抓取网页的网址或者说是链接）的优先队列, 由它来决定下一个要抓取的网址是什么, 同时去除重复的网址|
|下载器(Downloader)|用于下载网页内容, 并将网页内容返回给蜘蛛(Scrapy下载器是建立在twisted上的)|
|爬虫(Spiders)|爬虫是主要干活的, 用于从特定的网页中提取自己需要的信息, 即所谓的实体(Item)。用户也可以从中提取出链接,让Scrapy继续抓取下一个页面|
|项目管道(Pipeline)|负责处理爬虫从网页中抽取的实体，主要的功能是持久化实体、验证实体的有效性、清除不需要的信息。当页面被爬虫解析后，将被发送到项目管道，并经过几个特定的次序处理数据|
|下载器中间件(Downloader Middlewares)|位于Scrapy引擎和下载器之间的框架，主要是处理Scrapy引擎与下载器之间的请求及响应|
|爬虫中间件(Spider Middlewares)|介于Scrapy引擎和爬虫之间的框架，主要工作是处理蜘蛛的响应输入和请求输出|
|调度中间件(Scheduler Middewares)|介于Scrapy引擎和调度之间的中间件，从Scrapy引擎发送到调度的请求和响应|
 
Scrapy运行流程:

- 首先，引擎从调度器中取出一个链接(URL)用于接下来的抓取
- 引擎把URL封装成一个请求(Request)传给下载器，下载器把资源下载下来，并封装成应答包(Response)
- 然后，爬虫解析Response
- 若是解析出实体（Item）,则交给实体管道进行进一步的处理。
- 若是解析出的是链接（URL）,则把URL交给Scheduler等待抓取




## 悲剧啊!!
#### pyspider
运行起来pyspider不能跑爬虫，一执行就表示python已经奔溃用户你去吃屎吧。
官方文档有一个[常见问答](http://docs.pyspider.org/en/latest/Frequently-Asked-Questions/)
**Does pyspider Work with Windows？**
Yes, it **should**, some users have made it work on Windows. But as I don't have windows development environment, I cannot test. Only some tips for users who want to use >pyspider on Windows:
- Some package needs binary libs (e.g. pycurl, lxml), that maybe you cannot install it from pip, Windowns binaries packages could be found in http://www.lfd.uci.edu/~gohlke/pythonlibs/.
- Make a clean environment with virtualenv
- Try 32bit version of Python, especially your are facing crash issue.
- Avoid using Python 3.4.1 (#194, #217)

#### scrapy
官方文档，安装指南
Python 3 is not supported on Windows. This is because Scrapy core requirement Twisted does not support Python 3 on Windows.
你表妹啊，工作的电脑不用windows太不方便啊。
我就爱用64位操作系统和64位的python3你能把我怎么样？
我只能吃屎了。。

还是部署到vps上吧。。。

呵呵。人生苦短，少用windows。

本文链接: [http://ludaming.com/posts/spider/scrapy-and-pyspider.html](http://ludaming.com/posts/spider/scrapy-and-pyspider.html)