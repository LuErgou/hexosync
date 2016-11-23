---
title: peewee修改表结构(migration)
date: 2016-10-27 18:02:08
categories:
- Python
tags:
- Python
- sql
- peewee
---
使用peewee生成表模型后
难免会遇到要修改表的模型

<!-- more -->

## 概述
使用playhouse的migrate模块

```
from playhouse.migrate import *

# 这是postgresql的，实例化出一个migrator对象
my_db = PostgresqlDatabase(...)
migrator = PostgresqlMigrator(my_db)

# 使用migrate()执行一个或多个操作
# 添加了两个列，删除一个列
title_field = CharField(default='')
status_field = IntegerField(null=True)
 
migrate(
    migrator.add_column('some_table', 'title', title_field),
    migrator.add_column('some_table', 'status', status_field),
    migrator.drop_column('some_table', 'old_column'),
)
```

>Migrations are not run inside a transaction. If you wish the migration to run in a transaction you will need to wrap the call to migrate in a transaction block, e.g.
with my_db.transaction():
    migrate(...)


## 支持的操作

```
pubdate_field = DateTimeField(null=True)
comment_field = TextField(default='')
 

migrate(
    # 添加：指定表名，列名，表定义    
    migrator.add_column('table_name', 'pub_date', pubdate_field),
    migrator.add_column('table_name', 'comment', comment_field),

    # 删除： 指定表名，列名
    migrator.drop_column('story', 'some_old_field'),

    # 重命名列：指定表名，列名，新列名
    migrator.rename_column('story', 'mod_date', 'modified_date'),

    # 设置允许空或不允许空：表名，列名
    migrator.drop_not_null('story', 'pub_date'),      # 允许为空
    migrator.add_not_null('story', 'modified_date'),      # 非空

    # 重命名表：指定表名，新表名    
    migrator.rename_table('story', 'stories_tbl'),

    # 添加索引：表名，列名（集合），是否唯一
    migrator.add_index('story', ('pub_date',), False),    # 添加pub_date列索引，不唯一
    migrator.add_index('story', ('category_id', 'title'), True),    # 添加category_id, title 索引， 唯一

    # 删除索引： 表明，列名
    migrator.drop_index('story', 'story_pub_date_status')
    
)
```



本文链接[http://ludaming.com/posts/python/peewee-migration.html](http://ludaming.com/posts/python/peewee-migration.html)
我的博客: [ludaming.com](http://ludaming.com)
