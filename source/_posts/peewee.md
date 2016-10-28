---
title: peewee新手指南
date: 2016-10-27 13:53:57
categories:
- Python
tags:
- Python
- sql
- Python模块
- peewee
---

peewee是一个轻量级的ORM。
使用sqlalchemy内核，纯python编写，轻便。
支持sqlite，mysql，postgresql

<!-- more -->

例子使用的是postgresql（只有数据库的连接不同）
使用起来和django的orm感觉很相似。

## 安装
```
pip install peewee
```

## 模型

### 定义模型
```
from peewee import *
 
 
db = PostgresqlDatabase(
    'xxx',
    user='postgres',
    password='xxx',
    host='127.0.0.1',
)
 
class Person(Model):
    name = CharField()
    birthday = DateField()
    is_relative = BooleanField()
 
    class Meta:
        database = db
 
 
class Pet(Model):
    owner = ForeignKeyField(Person, related_name='pets')
    name = CharField()
    animal_type = CharField()
 
    class Meta:
        database = db  
```

### 执行
```
db.connect()
db.create_tables([Person, Pet])

```

### 保存数据

```
from datetime import date

# 生成对象并执行save操作
uncle_bob = Person(name='Bob', birthday=date(1960, 1, 15), is_relative=True)
uncle_bob.save() 

# 用create方法直接创建记录
grandma = Person.create(name='Grandma', birthday=date(1935, 3, 1), is_relative=True)
herb = Person.create(name='Herb', birthday=date(1950, 5, 5), is_relative=False)

# 嗯，grandma没有给姓
grandma.name = 'Grandma L'
grandma.save()

# 现在添加宠物
bob_kitty = Pet.create(owner=uncle_bob, name='Kitty', animal_type='cat')
herb_fido = Pet.create(owner=herb, name='Fido', animal_type='dog')
herb_mittens = Pet.create(owner=herb, name='Mittens', animal_type='cat')
herb_mittens_jr = Pet.create(owner=herb, name='Mittens Jr', animal_type='cat')
```
我有了一个bob叔叔，一个姓L的奶奶，还有一个不是亲戚不知道哪里来的逗逼herb。。。
他们还有宠物。

删除记录
```
herb_mittens.delete_instance()   # 返回值是删除的行数
```

### 查询数据
```
# 取一行数据，使用SelectQuery.get()
grandma = Person.select().where(Person.name == 'Grandma L').get()
print(grandma.birthday)

# 取所有数据
for person in Person.select():
    print(person.name, person.is_relative)
```


列出所有类型为cat的宠物的主人名字
```
query = Pet.select().where(Pet.animal_type == 'cat')
for pet in query:
    print(pet.name, pet.owner.name)
```
直接调用pet.owner.name可以得到想要的结果，但是由于原始的查询中没有查person表，peewee需要每次都查询person信息
会影响性能。
为了避免这种情况，我们应当在查询的时候直接同时查pet和person表，然后将两个表执行join
```
query = Pet.select(Pet, Person).join(Person).where(Pet.animal_type == 'cat')
for pet in query:
    print(pet.name, pet.owner.name)
```

列出Herb所有的宠物的名字, 并按照名字排序
```
query = Pet.select().join(Person).where(Person.name == 'Herb').order_by(Pet.name)
for pet in query:
    print(pet.name)
```

按照年龄降序列出名字
```
for person in Person.select().order_by(Person.birthday.desc()):
    print(person.name, person.birthday)
```


列出所有人的个人信息和宠物信息
```
for person in Person.select():
    print(person.name, person.pets.count(), 'pets')
    for pet in person.pets:
        print('    ', pet.name, pet.animal_type)
```
这和刚才情况一样，调用person.pets时peewee需要去查询一次pet表
依然来一个join

```
subquery = Pet.select(fn.COUNT(Pet.id)).where(Pet.owner == Person.id)
query = (Person
         .select(Person, Pet, subquery.alias('pet_count'))
         .join(Pet, JOIN.LEFT_OUTER)
         .order_by(Person.name))
 
for person in query.aggregate_rows():  # Note the `aggregate_rows()` call.
    print( person.name, person.pet_count, 'pets')
    for pet in person.pets:
        print( '    ', pet.name, pet.animal_type)
```


多条件查询
```
d1940 = date(1940, 1, 1)
d1960 = date(1960, 1, 1)
query = (Person
         .select()
         .where((Person.birthday < d1940) | (Person.birthday > d1960)))
for person in query:
    print(person.name, person.birthday)


query2 = (Person
         .select()
         .where((Person.birthday > d1940) & (Person.birthday < d1960)))
for person in query2:
        print(person.name, person.birthday)
```

## 其他
peewee也支持
group_by()
having()
limit() and offset()
详见[Querying](http://peewee.readthedocs.io/en/latest/peewee/querying.html#querying)


## 已有数据库？
peewee使用pwiz来生成模型
例如已有Person数据表
python -m pwiz -e postgresql Person > Person_models.py


## 修改表格式？
看[这个](http://ludaming.com/posts/python/peewee-migration.html)

本文链接[http://ludaming.com/posts/python/peewee.html](http://ludaming.com/posts/python/peewee.html)
我的博客: [ludaming.com](http://ludaming.com)

