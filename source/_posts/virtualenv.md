---
title: centos7 安装python3并配置virtualenv.md
date: 2016-09-07 09:00:57
categories:
- Python
tags:
- Python模块
- linux
---
{% cq %}virtualenv用来创建拥有独立安装目录的隔离python环境。{% endcq %}

<!-- more -->
正常情况下，我们写的多个python应用可能依赖不同版本的python或者某个包。
所以为每个python应用都创建一个独立的python环境是有必要的
这就要用到virtualenv


## 注意
通过源码编译安装python3的时候，一定要安装好编译环境。
我就是没有装sqlite-devel，造成python3无法Import sqlite3
如果出现了这个情况，就删除python3目录，安好变异环境后再重新编译安装python3


## 安装python3
centos7默认没有python3，为了能建立python3的隔离环境，需要先为系统安装python3
```
# 准备编译环境
>>>yum groupinstall 'Development Tools'
>>>yum install zlib-devel bzip2-devel openssl-devel ncurese-devel
# 安装python3
>>>wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tar.xz
>>>tar xvf Python-3.5.2.tar.xz
>>>cd Python-3.5.2

>>>mkdir /usr/python3.5
>>>./configure --prefix=/usr/python3.5
>>>make & make install
# 建立连接
>>>ln -s /usr/python3.5/bin/python3 /usr/bin/python3
>>>ln -s /usr/python3.5/bin/pip3 /usr/bin/pip3
```
这样通过python命令来执行python2.7，通过python3命令来执行python3.5
系统中原有的python2.7不要动,否则会导致依赖python2的命令无法使用。
通过pip 安装的模块，执行文件会在/usr/bin/中。
通过pip3 安装的，执行文件会在/usr/python3.5/bin/中。


## 安装virtualenv
```
>>>pip install virtualenv
Collecting virtualenv
Downloading virtualenv-15.0.3-py2.py3-none-any.whl (3.5MB)
100% |################################| 3.5MB 197kB/s
Installing collected packages: virtualenv
Successfully installed virtualenv-15.0.3
```
## 创建虚拟环境
```
>>>virtualenv -p /usr/bin/python3 ~/envs/testenv
Running virtualenv with interpreter /usr/bin/python3
Using base prefix '/usr/python3.5'
New python executable in /root/envs/hcomic3/bin/python3
Also creating executable in /root/envs/hcomic3/bin/python
Installing setuptools, pip, wheel...done.
```

```
# 启动虚拟环境
>>>source ~/envs/testenv/bin/active

# 退出环境
>>>deactivate
```
为多个应用配置各自的虚拟环境，便于管理。
每次修改或运行某个应用的时候，只用开启对应的虚拟环境即可。


## virtualenv参数
|参数|解释|
|----|----|
|--version|显示版本|
|-p PYTHON_EXE or --python=PYTHON_EXE|选择python编译器（系统里同时存在2和3的需要手动设定）|
|--no-site-packages|已过时。创建出的环境不使用系统环境中安装过的包，现在默认不使用系统环境中的包|
|--system-site-packages|创建出的环境使用系统环境中的包|
更多参数用-h查看即可


本文链接: [http://ludaming.com/posts/Python/virtualenv.html](http://ludaming.com/posts/Python/virtualenv.html)