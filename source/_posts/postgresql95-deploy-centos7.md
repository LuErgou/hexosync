---
title: postgresql9.5 安装配置和使用（on centos7）
date: 2016-09-09 16:32:09
categories:
- postgresql
tags:
- sql
- postgresql
- 数据库
---


从mysql被oracle收购以后，centos已经使用mariadb来替换mysql了。
PostgreSQL逐渐成为开源关系型数据库的首选。

本文介绍PostgreSQL的安装和基本用法。基于CentOS7。
<!-- more -->

## 安装
看了一下，centos7自带源上使用的9.2版本的postgresql
官网稳定版目前是9.5.4
决定使用9.5.4版本（其实就使用9.2版本就可以了）

从[这里](http://yum.postgresql.org/repopackages.php)huode 获取rpm包
选择你需要的版本右键复制连接即可
```
# 安装源
yum install https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm
# 安装postgresql
yum install postgresql95-server postgresql95-contrib posgresql95
# 初始化数据库
/usr/pgsql-9.5/bin/postgresql95-setup initdb
# 启动
systemctl start postgresql-9.5.service
```
为了避免和默认9.2版本冲突，所以用postgresql给出的源安装后，命令是postgresql-9.5



## 用户配置
执行完初始化操作后
postgresql会自动创建生成两个用户和一个数据库
- linux系统用户postgres：用户管理数据库的系统用户
- postgresql用户postgres：数据库超级管理员用户
- 数据库postgres：用户postgres的默认数据库

这两个用户密码都是自动生成的。
如果自己本地使用，直接修改这两个用户的密码，直接用就好了。

为了安全考虑我们应该专门再建立一个postgre用户和相应的linux用户
然后给予新建用户一个数据库。
```
# 创建一个linux系统用户dbuser
adduser dbuser
# 使用数据库管理员用户登录psql来新建以及配置新用户
# 先切换到postgres用户
su postgres
# 执行psql命令登录psql控制台（此时不需要密码，原因在后面说）
psql
# 此时会进入到控制台（系统提示符变为'postgres=#'）
# 先为管理员用户postgres修改密码
\password postgres
# 建立新的数据库用户（和之前建立的系统用户要重名）
create user dbuser with password 'pwd';
# 为新用户建立数据库
create database testdb owner dbuser;
# 把新建的数据库权限赋予dbuser
grant all privileges on database testdb to dbuser;
# 退出控制台
\q
```
**注意**
尼玛在控制台下新建用户新建库之类的命令一定记得加分号
没有分号执行后没有任何反映，我还以为是成功了（最好的消息不应该就是没有消息嘛？）
执行成功会返回
```
CREATE ROLE
CREATE DATABASE
GRANT
```

## 登录数据库
```
psql -U dbuser -d testdb -h 127.0.0.1 -p 5432
```

参数含义如下：-U指定用户，-d指定数据库，-h指定服务器，-p指定端口。
输入上面命令以后，系统会提示输入dbuser用户的密码。输入正确，就可以登录控制台了。
 
**root用户，执行了结果是psql: FATAL:  Ident authentication failed for user "dbuser"**
**su 切换到对应用户可以登录**
**远程工具连接数据库也报错，no pg_hba.conf entry for host xxx ...**
为嘛？！
认证权限的问题
见下面psql认证权限配置


**psql命令存在简写形式**
如果当前Linux系统用户，同时也是PostgreSQL用户，则可以省略用户名（-U参数的部分）。
如果使用dbuser登录系统，PostgreSQL数据库存在同名用户dbuser，可以直接使用下面的命令登录数据库，且不需要密码。
```
psql testdb
```
此时，如果PostgreSQL内部还存在与当前系统用户同名的数据库，则连数据库名都可以省略。比如，假定存在一个叫做dbuser的数据库，则直接键入psql就可以登录该数据库。
```
psql
```

## psql配置
### 开启远程访问
psql默认只能通过本地访问。
打开配置文件  /var/lib/pgsql/9.5/data/postgresql.conf
将listen_address设置为'*'
重启psql

### psql认证权限配置
认证权限配置文件  /var/lib/pgsql/9.5/data/pg_hba.conf
psql 支持11中身份验证方式
下面是常见的四种
- trust
    凡是连接到服务器的，都是可信任的。只需要提供psql用户名，可以没有对应的操作系统同名用户。
- password 和 md5
    对于外部访问，需要提供psql用户名和密码。
    对于本地连接，提供psql用户名密码之外，还需要有操作系统访问权。（用操作系统同名用户验证）
    password和md5的区别就是外部访问时传输的密码是否用md5加密。
- ident
    对于外部访问，从ident服务器获得客户端操作系统用户名，然后把操作系统作为数据库用户名进行登录
    对于本地连接，实际上使用了peer
- peer
    通过客户端操作系统内核来获取当前系统登录的用户名，并作为psql用户名进行登录。
    
**centos中默认本地认证方式是ident，也就是peer方式**
psql用户名必须有对应的操作系统用户名。并且必须亿操作系统同名用户登录linux才可以登录psql。
想用其他用户（例如root）登录psql，修改本地认证方式为trust或者password即可。

修改/var/lib/pgsql/9.5/data/pg_hba.conf文件

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            password
host    all             all             0.0.0.0/0            md5
# host    all             all             127.0.0.1/32            ident
...
```
重启postgresql


## psql控制台命令

### psql命令
- \h：查看SQL命令的详细解释，例如 \h select
- \?：查看psql命令列表
- \l：列出所有数据库
- \c [database_name]：连接其他数据库
- \d ：列出数据库的所有表
- \du：列出所有数据库用户
- \conninfo：列出连接

### 导出数据库
pg_dump -h localhost -U postgres postgres > ~/test.sql
### 导入数据库
psql -U postgres postgres < ~/test.sql


## 数据库操作
sql语言
```
# 创建表
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);
 
# 插入
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');
 
# 选择
SELECT * FROM user_tbl;
 
# 更新
UPDATE user_tbl set name = '李四' WHERE name = '张三';
 
 # 删除
DELETE FROM user_tbl WHERE name = '李四' ;
 
# 添加列
ALTER TABLE user_tbl ADD email VARCHAR(40);
 
# 更新列
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;
 
# 更名列
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;
 
# 删除列
ALTER TABLE user_tbl DROP COLUMN email;
 
# 重命名表
ALTER TABLE user_tbl RENAME TO backup_tbl;
 
# 删除表
DROP TABLE IF EXISTS backup_tbl;
```



本文链接: [http://ludaming.com/posts/postgresql/postgresql95-deploy-centos7.html](http://ludaming.com/posts/postgresql/postgresql95-deploy-centos7.html)



