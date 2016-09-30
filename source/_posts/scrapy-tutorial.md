---
title: scrapy1.1(一) 新手指南(python3)
date: 2016-09-23 14:21:08
categories:
- spider
tags:
- Python
- spider
- Python框架
---


初次接触scrapy
记录一下官方[tutorial]（http://doc.scrapy.org/en/1.1/intro/tutorial.html）
官方指南中爬取的网站是[quotes.toscrape.com](quotes.toscrape.com)

<!-- more -->
 
指南中将会完成：
- 新建一个项目
- 定义需要爬取的内容
- 写一个爬虫去爬取网站
- 使用命令行导出爬取到的内容



## 部署
我是在自己的vps（centos7）上部署的，使用了virtualenv，使用的python版本为3.5.2
注意编译环境一定要装好。
关于centos7安装python3.5，启用virtualenv以及必须的编译环境，请看[这里](http://ludaming.com/posts/Python/virtualenv.html)
 
```
pip install scrapy
```
安装结束。。
 
 如果出现
no package libffi 错误
```
yum install libffi-devel
```

## scrapy 新手指南
### 建立一个新项目
在目录下执行
'''
scrapy startproject tutorial
'''
在当前目录下会生成一个名为tutorial的文件夹
该文件夹结构如下

    tutorial/
        scrapy.cfg            # scrapy的部署配置文件

        tutorial/             # 项目的python块，我们的代码就放到这里
            __init__.py

            items.py          # 项目的目标文件（确定要爬取的目标）

            pipelines.py      # 项目的管道文件（确定存储内容和方式）

            settings.py       # 项目的配置文件
    
            spiders/          # 项目的爬虫文件
                __init__.py

### 定义目标文件（Items）
Items是用来加载爬取内容的容器，和Python中的字典有些相似，同时提供额外的功能
可以配合Item Loaders使用（目前不用）
声明Item只需要建立一个 scrapy.Item 类并且将它的属性设置为scrapy.Field 对象。
有点像使用ORM。（不知道ORM没有关系，现在用不到）

我们来抓取[quotes.toscrape.com](quotes.toscrape.com)中的正文和作者
编辑tutorial/items.py文件, 添加一个类

```
import scrapy

class QuoteItem(scrapy.Item):
    text = scrapy.Field()
    author = scrapy.Field()
```
### 爬虫文件（spiders）
#### 爬取网页代码
spiders是我们定义的用来爬取所需内容的文件
建立新的爬虫文件，需要继承scrapy.Spider，定义以下属性
- name 爬虫名称，同一个项目中不能有重复名称的spider。
- start_url 起始url列表，执行后每个url会返回一个Response对象
- parse()调用的时候传入从每一个 URL 传回的 Response 对象作为参数，负责解析并匹配抓取的数据(解析为 item)，跟踪更多的 URL。
新建tutorial/spiders/quote_spider.py文件
```
import scrapy
 
 
class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    # 此时只是将页面代码下载下来并保存，没有进行分析 
    def parse(self, response):
        filename = 'quotes-' + response.url.split("/")[-2] + '.html'
        with open(filename, 'wb') as f:
            f.write(response.body)
```
执行后（此时我们只定义了类，想要执行就在项目根目录下执行scrapy crawl quotes）：
Scrapy为每个url建立scrapy.Request对象
执行每个requests
将返回的response对象当作参数传给parse()方法。

#### 分析并获取所需内容Scrapy Selectors
当页面代码下载到本地后，我们需要对数据进行分析和提取
Scrapy提供Selector类来进行数据获取，详细信息见[介里](http://doc.scrapy.org/en/1.1/topics/selectors.html#topics-selectors)
Selectors 支持一下四种方式
- xpath(): 返回一个selector列表，每一项表示一个 xpath 参数表达式选择的节点
- css(): 返回一个selector列表，每一项表示一个 css 参数表达式选择的节点
- extract(): 返回一个字符串列表，为使用正则表达式抓取出来的内容
- re(): 返回一个字符串列表，为使用正则表达式抓取出来的内容

XPath 详细信息见[XPath 教程](http://www.w3school.com.cn/xpath/)

修改tutorial/spiders/quote_spider.py文件
```
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]
 
    def parse(self, response):
        for quote in response.xpath('//div[@class="quote"]'):
            text = quote.xpath('span[@class="text"]/text()').extract_first()    
            author = quote.xpath('span/small/text()').extract_first()    
            print(u'{}: {}'.format(author, text))
```
parse()：
取每一个class为quote的div，遍历
在div中去text和author的正文。
extract_first(): 返回selector列表的第一项。

### 使用定义好的目标文件（Item）


```
import scrapy
from tutorial.items import QuoteItem
 
 
class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]
 
    def parse(self, response):
        for quote in response.xpath('//div[@class="quote"]'):
            item = QuoteItem()
            item['text'] = quote.xpath('span[@class="text"]/text()').extract_first()
            item['author'] = quote.xpath('span/small/text()').extract_first()
            yield item

```
有关yield，见[这里](http://ludaming.com/posts/Python/python-generator.html)
### 爬取后续连接（following links）
前面的代码只会爬取我们在start_urls中定义的两个页面
而爬虫应当从start_url中爬取到的内容中找到后续的url链接
```
import scrapy
from tutorial.items import QuoteItem
 
class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]
 
    def parse(self, response):
        # 爬取所需内容
        for quote in response.xpath('//div[@class="quote"]'):
            item = QuoteItem()
            item['text'] = quote.xpath('span[@class="text"]/text()').extract_first()
            item['author'] = quote.xpath('span/small/text()').extract_first()
            yield item
        next_page = response.xpath('//li[@class="next"]/a/@href').extract_first()
        # 如果找得到下一页的连接，创建一个request并且设定这个request的callback为parse（）
        if next_page:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```
### 存储爬取到的信息
#### 使用Feed exports
如果对爬取到的信息不需要进行额外处理（例如保存到数据库），可以通过scrapy命令直接输出json等格式的文件，详见[Feed exports](http://doc.scrapy.org/en/1.1/topics/feed-exports.html#topics-feed-exports)
例如输出为json文件
```
# 执行后会生成items.json文件
scrapy crawl quotes -o items.json
```

#### 使用管道pipelines
新建项目的时候，会自动生成管道文件tutrial/pipelines.py ，
如果对爬取到的信息进行额外处理，可以使用pipelines

## 总结
话说指南到这里就结束了
感觉很不给力啊
pipelines 不提啦？
只提到了用命令行执行。。。
scrapy.cfg文件呢？
settings.py文件呢？

好吧，只是解释了一下
- 新建一个项目
- 定义需要爬取的内容
- 写一个爬虫去爬取网站
- 使用命令行导出爬取到的内容



更多的内容在之后的文档里。
今天就先记录到这里好了。


本文链接[http://ludaming.com/posts/spider/scrapy-tutorial.html](http://ludaming.com/posts/spider/scrapy-tutorial.html)
我的博客: [ludaming.com](http://ludaming.com)
