---
title: selenium3.0.1使用chrome，设置代理ip进行爬取
date: 2016-11-23 15:58:34
categories:
- spider
tags:
- spider
- Python
---
工作需要爬取淘宝信息
发现淘宝宝贝的评论，以及天猫淘宝店铺的信用信息页面中的数据都是加密了的

今年上半年爬取的时候还是可以的，直接分析js即可
现在需要写cookie，只好使用selenium

本来使用的是phantomjs，但是没法设置代理ip。爬取的多了会被淘宝封锁ip


前两天网上也看了几个文章，本来想用firefox，结果咋的都不成功，决定看着官网文档使用chrome
申请了一台电脑，用selenium+chrome来爬取信息
<!-- more -->

## 安装selenium
```
pip install selenium
```
版本是3.0.1

## 安装chrome和chromeDriver
安装chrome就不说了
chromeDriver去[这里](https://sites.google.com/a/chromium.org/chromedriver/downloads)下载
注意chromeDriver的版本，要选择支持你本机chrome版本的chromeDriver

尼玛。。只有win32版本的chromeDriver，可我是64bit的win7和64bit的chrome呀
（没关系，好用。。）

下载好chromedriver.exe后，把它放到chrome目录下。（和chrome.exe一个目录）
然后在系统环境变量中把chrome目录放到path里（也可以不放，在代码中指名executable_path即可）

```
from selenium import webdriver
driver = webdriver.Chrome(executable_path="C:\\Program Files (x86)\\Google\\Chrome\\Application\\chromedriver.exe")
driver.get(taobao_url)                        # 访问淘宝宝贝页面，获取cookie
driver.get(taobao_comment_url)        # 直接访问宝贝评论会被反爬虫检测到。上一步获得cookie后可得到评论数据
print(driver.find_element_by_xpath('/html/body').text)
driver.quit()
```


>webdriver退出有两种，一种是close，一种是quit
close是只关闭当前标签，并且不清除缓存。使用quit是关闭浏览器。
如果爬取数量多，记住使用quit

## 使用代理
爬取淘宝信息，不实用代理就是作死啊。
之前有次害的公司上淘宝全都得登录。。。
```
from selenium import webdriver
PROXY = "124.206.133.227:80"
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server={0}'.format(PROXY))
chrome = webdriver.Chrome(executable_path='C:\\Program Files (x86)\\Google\\Chrome\\Application\\chromedriver.exe', chrome_options=chrome_options)
chrome.get('http://1212.ip138.com/ic.asp')
print('2: ', chrome.page_source)
# chrome.quit()

```

## 判断代理失效
代理有很多失效的，执行get后会显示网页无法连接
```
PROXY = "61.168.162.32:80"
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server={0}'.format(PROXY))
chrome = webdriver.Chrome(executable_path='C:\\Program Files (x86)\\Google\\Chrome\\Application\\chromedriver.exe', chrome_options=chrome_options)
 
chrome.get(taobao_url)
if '未连接到互联网' in chrome.page_source:
    print('代理不好使啦')
if 'anti_Spider-checklogin&' in chrome.page_source:
    print('被anti_Spider check啦')
```
自己写处理方法吧~~啦啦啦啦~~~





本文链接[http://ludaming.com/posts/spider/selenium-chrome.html](http://ludaming.com/posts/spider/selenium-chrome.html)
我的博客: [ludaming.com](http://ludaming.com)
