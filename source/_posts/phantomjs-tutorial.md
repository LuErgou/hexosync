---
title: phantomjs2.1 初体验
date: 2016-09-27 13:53:57
categories:
- spider
tags:
- spider
---
上次看了一下scrapy1.1的新手指南

决定写个小爬虫实验一下

目标网站是[http://www.dm5.com/manhua-huofengliaoyuan](http://www.dm5.com/manhua-huofengliaoyuan)
准备爬取漫画火凤燎原的已有章节，将图片保存到本地

开始一切顺利，从漫画目录页面开始，拿到了每一话（卷）的地址

然而访问后发现页面中的图片地址都是加过密的，找不到真实地址。
（网页源代码中没有，f12看network大概看了一下，也没有发现地址，所以应该是加密了）

后来试着打开了几个漫画网站，只有腾讯动漫能在页面中找到图片的连接。

决定试一试phantomjs
>PhantomJS是一个无界面的,可脚本编程的WebKit浏览器引擎。它原生支持多种web 标准：DOM 操作，CSS选择器，JSON，Canvas 以及SVGPhantomJS是一个无界面的,可脚本编程的WebKit浏览器引擎。它原生支持多种web 标准：DOM 操作，CSS选择器，JSON，Canvas 以及SVG。
<!-- more -->

以下使用火凤燎原第5卷第2页的地址[http://www.dm5.com/m5342-p2/](http://www.dm5.com/m5342-p2/)为例子
按着官网指南走一边。

## 安装phantomjs2.1.1
从[这里](http://phantomjs.org/download.html)下载对应好的包
或者自己下载源码进行编译(据说自己编译巨慢)

我下载的是phantomjs-2.1.1-linux-x86_64.tar.bz2
解压缩，在目录里执行
```
./bin/phantomjs -v
-->2.1.1
```
话说这样执行没问题，也给了+x权限，~~但是ln -s 到/usr/bin目录下却无法执行。。~~
将phantomjs连接到python环境的bin目录下。




## 新手指南
官方[新手指南](http://phantomjs.org/quick-start.html)
我是按着官方走，但是实验自己需要爬取的页面

### Hello,World!
新建一个文档hello.js
内容：
```
# 在控制台打印日志
console.log('Hello, world!');
// 退出phantomjs
phantom.exit();
```
在hello.js目录下执行
```
phantomjs hello.js
--> Hello,world!
```
注意任何时候都不要忘记**phantom.exit();**
他用来让phantomjs退出

### 载入页面并将整个页面保存成图片
```
//创建一个webpage对象
var page = require('webpage').create();
// 打开页面
page.open('http://www.dm5.com/m5342-p2/', function(status) {
  // 输出状态
  console.log("Status: " + status);
  if(status === "success") {
    // 如果状态为success，将整个page保存为hfly.jpg（也可以是png，pdf, gif）
    page.render('hfly.jpg');
  }
  phantom.exit();
});
```

执行代码后
![http://oc1suuvql.bkt.clouddn.com/image/3f0da110-75fa-44be-b504-9f0b01a0dc89.jpg](http://oc1suuvql.bkt.clouddn.com/image/3f0da110-75fa-44be-b504-9f0b01a0dc89.jpg)


 
感觉速度很慢，不过这是保存了整个页面，如果只保存图片会不会快一点？
官网真贴心，紧跟着就给了一个测速脚本

### 测试速度loadspeed.js
这个速度是phantomjs加载页面并且渲染页面的总耗时。

```
var page = require('webpage').create(),
  system = require('system'),
  t, address;
 
if (system.args.length === 1) {
  console.log('Usage: loadspeed.js <some URL>');
  phantom.exit();
}
 
t = Date.now();
// 读取控制台参数
address = system.args[1];
page.open(address, function(status) {
  if (status !== 'success') {
    console.log('FAIL to load the address');
  } else {
    t = Date.now() - t;
    console.log('Loading ' + system.args[1]);
    console.log('Loading time ' + t + ' msec');
  }
  phantom.exit();
});
```
执行，参数为要测试的地址
**地址前面要加上http://或者https://**
```
phantomjs loadspeed.js http://www.baidu.com
-->Loading time 5403 msec
phantomjs loadspeed.js http://www.dm5.com/m5342-p2/
-->Loading time 76584 msec
```
。。。
敢问你是不是在逗我？秒速五厘米很美、、秒速76厘米就是逗逼了。。
为什么这么慢？他怎么就这么慢？是我环境的问题嘛？
（因为我是用的虚拟机centos7，本地win7 pycharm remote evns，虚拟机的内存小cpu只分配了一个核心，可能渲染的就慢一些？）

~~早有耳闻phantomjs速度慢，没想到google诚不欺我。。~~
不行。。我要在本地win7上也试一下。
本地
```
phantomjs.exe loadspeed.js http://www.baidu.com
-->Loading time 701msec
phantomjs.exe loadspeed.js http://www.dm5.com/m5342-p2/
-->Loading time 5534 msec
```
我猜对了。。。
那我真是好尴尬，本地win7 安装python3 的各种包真是无力
搞个虚拟机专门用来配置python环境。。~~没想到phantomjs需要一定的性能。。~~

我再试试我的vps。。512M，1核，本地虚拟机是1核，768M。
```
phantomjs.exe loadspeed.js http://www.google.com
-->Loading time 415msec
phantomjs.exe loadspeed.js http://www.nba.com
-->Loading time 2800 msec
phantomjs.exe loadspeed.js http://www.dm5.com/m5342-p2/
-->Loading time 7993 msec
```
看来不是性能问题，是下载速度。。
我用的虚拟机是本地桥接，没想到网速会差这么多

!!又尼玛反转，剧情神打脸，啪啪啪啪
![http://oc1suuvql.bkt.clouddn.com/image/ccf06c6b-c876-4f45-a194-ba04e5eea016.jpg](http://oc1suuvql.bkt.clouddn.com/image/ccf06c6b-c876-4f45-a194-ba04e5eea016.jpg)


虚拟机的网速也是呱呱叫的
目前，唯一能解释的就是vmware不行，效率低（本身就是虚拟机）
但是vps也是虚拟机来着。。到底人家专业做的么？


小结：使用phantomjs，要么在本地机器上运行，要么直接放vps上运行。
用vmware，慢出个翔、、
 

### 代码评估（code evaluation）
不知道怎么翻译，明明就是分析代码然后取出需要的内容

使用evaluate()方法去审核页面代码，这个方法是沙盒式的，不会去执行页面外的js代码。
（也就是说在外面定义的变量不能在evalute方法中使用）
**evalute方法是直接去操作page对象了，把处理后取得的内容返回。**

evaluate()方法返回一个对象，也只能返回一个对象，不能包含函数。

```
// 获得页面标题
url = 'http://www.dm5.com/m5342-p2/'
// 创建webpage对象
var page = require('webpage').create();
page.open(url, function(status) {
  if(status === "success"){
    // 从page对象里，evalute出来title（不知道怎么翻译。。）
    var title = page.evaluate(function() {
      return document.title;
    });
    console.log('Page title is ' + title);
  }else {
    console.log('failed');
  }
  phantom.exit();
});
```
执行后输出页面标题

通常一个网页自己也有console输出，在chrome里f12选择console就能看到。
这个输出在phantomjs里是不会被显示出来的
evaluate方法里的console.log也不会显示（因为相当于网页自己的console输出了。）
如果想要输出
需要重写onConsoleMessage方法

```
url='http://www.dm5.com/m5342-p2/'
var page = require('webpage').create();
page.onConsoleMessage = function(msg) {
  console.log('Page title is ' + msg);
};
page.open(url, function(status) {
  page.evaluate(function() {
    console.log(document.title);
  });
  phantom.exit();
});
```

我尼玛。
为啥会有一个百度的广告。
估计是这个网站使用了百度推广或者统计吗？
但是chrome里看控制台，却没有这个百度的输出
 不知道为啥

只要phantomjs脚本运行起来，就可以使用css 选择器和标准的js DOM 操作。
所以需要了解一下css selector 和 DOM 操作

### 自动化任务
由于phantomjs可以加载和操作web界面，所以可以用它来做自动化。
详见[page automation tasks](http://phantomjs.org/page-automation.html)

### 网络监控
歪了歪了，，我只想拿页面上的图片。。
但是官方指南只是介绍大概功能，鉴于这是最后一点内容。也一并记录下来好了

phantomjs 有网络通信的检查功能，很适合用来做网络行为的分析。
当接受到请求时，可以通过重写onResourceRequested和onResourceReceived回调函数来实现接收到资源请求和资源接受完毕的监听。
详见[network monitoring](http://phantomjs.org/network-monitoring.html)
```
// 网络监听
url='http://www.dm5.com/m5342-p2/'
var page = require('webpage').create();
page.onResourceRequested = function(request) {
  console.log('Request ' + JSON.stringify(request, undefined, 4));
};
page.onResourceReceived = function(response) {
  console.log('Receive ' + JSON.stringify(response, undefined, 4));
};
page.open(url);
```
这样在webpage载入的时候，会把所有网络请求的请求内容和响应内容打印到控制台。

### 官方例子
里面有很多样例代码，可以直接看，不同的再去翻手册
[Examples](http://phantomjs.org/examples/index.html)

## 记录一下拿图片地址的代码
果然官方指南只是大略的介绍一下

拿图片地址的文件get_pic_url.js
```
// 接受漫画网页地址
url = system.args[1];
var page = require('webpage').create();
page.open(url, function(status) {
  if(status === "success"){
            // 处理页面
          var pic_url = page.evaluate(function() {
            // DOM操作
            return document.getElementById('cp_image').getAttribute('src');
          });
          console.log(pic_url);
  }else{
    console.log('failed');
  }
  phantom.exit();
});
```
执行 phantomjs get_pic_url.js http://www.dm5.com/m5342-p2/
输出图片地址。


## 总结
phantomjs是一个无界面的浏览器。
所以在代码最后一定要加入phantom.exit()方法确保其退出。

phantomjs加载页面以后
用evaluate()方法对渲染后的页面代码进行操作，此函数值操作页面，和我们执行的js不在一个域。


phantomjs可以做自动化测试，网络监听等工作。


。。可是怎么和python交互啊？
改天再看看

本文链接[http://ludaming.com/posts/spider/phantomjs-tutorial.html](http://ludaming.com/posts/spider/phantomjs-tutorial.html)
我的博客: [ludaming.com](http://ludaming.com)
