---
title: pyspider部署以及遇到的问题（on centos7）
date: 2016-09-08 14:13:10
categories:
- spider
tags:
- spider
- Python
- Python框架
- 编程
---
{% cq %}人生苦短，少用windows{% endcq %}

<!-- more -->

我是在自己的vps（centos7）上部署的，使用了virtualenv，使用的python版本为3.5.2
注意编译环境一定要装好。
关于centos7安装python3.5，启用virtualenv以及必须的编译环境，请看[这里](http://ludaming.com/posts/Python/virtualenv.html)

## 部署
```

# 新建虚拟环境并进入
>>>virtualenv -p /usr/bin/python3 ~/envs/testenv
>>>source ~/envs/testenv/bin/activate

# 安装pycurl（安装pyspider时就会自动安装，但是自动安装的在我这儿出错）
>>>export PYCURL_SSL_LIBRARY=nss
>>>pip install pycurl --no-cache-dir

# 安装pyspider
>>>pip install pyspider

# 执行
pyspider
```


## 遇到的问题
### pip install pyspider 执行后提示curl安装不上
    是因为编译环境没有装好，yum安装libcurl-devel即可
### 安装好pyspider成功，运行pyspider命令时报错
```
RuntimeError: Click will abort further execution because Python 3 was configured to use ASCII as encoding for the environment.  Either run this under Python 2 or consult http://click.pocoo.org/python3/ for mitigation steps.
```
是因为系统编码不对，按照[http://click.pocoo.org/python3/](http://click.pocoo.org/python3/)上的说明
>You are dealing with an environment where Python 3 thinks you are restricted to ASCII data. The solution to these problems is different depending on which locale your computer is running in. 
If you are on a US machine, en_US.utf-8 is the encoding of choice. On some newer Linux systems, you could also try C.UTF-8 as the locale:
export LC_ALL=C.UTF-8
export LANG=C.UTF-8

执行
```
export LC_ALL=en_US.utf-8
export LANG=en_US.utf-8
```

###运行pyspider命令时报错
```
libcurl link-time ssl backend (nss) is different from compile-time ssl backend (none/other)
```
卸载pycurl，按照之前的说明，重新安装。
```
pip uninstall pycurl
export PYCURL_SSL_LIBRARY=[nss|openssl|ssl|gnutls]
pip install pycurl --no-cache-dir
```
[]中取决于你报错时link-time ssl backend后面括号的内容。保持相同即可
关于这部分讨论的stackoverflow的页面，[http://stackoverflow.com/questions/21096436/ssl-backend-error-when-using-openssl](http://stackoverflow.com/questions/21096436/ssl-backend-error-when-using-openssl)

###运行pyspider命令时提示**ImportError: No module named '_sqlite3'**
这是python3没有编译好。见[这里](http://ludaming.com/posts/Python/virtualenv.html)

## 运行后效果
```
[W 160907 13:37:27 run:403] phantomjs not found, continue running without it.
[I 160907 13:37:29 result_worker:49] result_worker starting...
[I 160907 13:37:30 processor:208] processor starting...
[I 160907 13:37:30 scheduler:569] scheduler starting...
[I 160907 13:37:30 scheduler:508] in 5m: new:0,success:0,retry:0,failed:0
[I 160907 13:37:30 tornado_fetcher:508] fetcher starting...
/root/envs/hcomic/lib/python3.5/site-packages/flask/exthook.py:71: ExtDeprecationWarning: Importing flask.ext.login is deprecated, use flask_login instead.
  .format(x=modname), ExtDeprecationWarning
[I 160907 13:37:30 scheduler:683] scheduler.xmlrpc listening on 127.0.0.1:23333
[W 160907 13:37:31 app:61] WebDav interface not enabled: ImportError("No module named 'wsgidav'",)
[I 160907 13:37:31 app:76] webui running on 0.0.0.0:5000
```

此时开放vps的5000端口，就可以通过浏览器访问pyspider的webui了


pyspider默认没有phantomjs和wsgidav。
需要自己配置开启。
默认使用sqlite数据库，会在执行pyspider命令的目录下生成data目录。


本文链接: [http://ludaming.com/posts/spider/pyspider-deploy-on-centos7.html](http://ludaming.com/posts/spider/pyspider-deploy-on-centos7.html)


