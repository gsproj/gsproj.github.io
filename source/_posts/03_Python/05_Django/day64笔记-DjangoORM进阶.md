---
title: Day64-DjangoORM进阶
date: 2022-09-15 14:44:22
categories:
- Python
- 09_Django
tags:
---

“Day64 Django ORM 操作数据库 进阶用法 学习笔记”



# 今日内容概要(重要)

模型层(ORM语法):跟数据库打交道的

* 单表查询(增删改查)
* 常见的十几种查询方法
* 神奇的双下划线查询
* 多表操作
* 外键字段的增删改查
* 跨表查询(重点)
  * 子查询
  * 联表查询
* 聚合查询
* 分组查询
* F与Q查询

# 一、环境准备

准备实验环境

## 1.1 基础准备

1、创建数据库`day64`

```mysql
# 修改root密码:
mysql> use mysql;
Database changed
mysql> alter user 'root'@'localhost' identified by '123123';
Query OK, 0 rows affected (0.01 sec)

# 创建数据库
mysql> create database day64;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| day61              |
| day64              |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
7 rows in set (0.00 sec)
```

2、创建Django项目

3、设置`setting.py`，连接Mysql的参数

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'day64',
        'USER': 'root',
        'PASSWORD': '123123',
        'HOST': '127.0.0.1',
        'PORT': 3306,
        'CHARSET': 'utf8',
    }
}
```

4、设置`__init__.py`文件，导入pymysql

```python
import pymysql
pymysql.install_as_MySQLdb()
```

5、创建app01应用，并在`settings.py`中注册应用

```shell
python manage.py startapp app01

# setting.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01'
]

```

6、编辑`app01`中的`models.py`，创建ORM模型

```python
from django.db import models


# Create your models here.
class User(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    register_time = models.DateField()


'''
    DateFiled
    DateTimeFiled
        两个重要参数
        auto_now : 每次操作数据的时候，该字段会自动将时间更新
        auto_now_add: 在创建数据的时候会自动将当前创建时间记录下来，只要不人为的改变，就一直不变
'''
```

7、执行创建表操作

```shell
# 本地记录
python manage.py makemigrations
# 同步到数据库
python manage.py migrate
```

## 1.2 测试脚本

>🌟 当你只是想测试django中的某一个py文件内容，那么你可以不用书写前后端交互的形式而是直接写一个测试脚本即可，脚本代码无论是写在应用下的tests.py还是自己单独开设py文件都可以

编辑`test.py`文件

```python
from django.test import TestCase

# Create your tests here.

import os
import sys


if __name__ == '__main__':
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", 'day64_ORM进阶.settings')
    import django
    django.setup()
    # 在这个代码块的下面就可以测试django里面的单个py文件了
    # 所有的代码必须等环境准备完毕之后才能书写

    from app01 import models
    models.User.objects.all()
```



# 二、单表操作

单表的ORM操作

PS: 案例中的代码都是写在`test.py`中执行

## 2.1 增删改查

### 2.1.1 增

增有两种方式

```python
# 方式一
res = models.User.objects.create(name='jason', age=18, register_time='2022-09-27')
print(res)

# 方式二
import datetime
ctime = datetime.datetime.now()
user_obj = models.User(name='tank', age=38, register_time=ctime)
user_obj.save()
```

查看结果：

```mysql
mysql> select * from app01_user;
+----+-------+-----+---------------+
| id | name  | age | register_time |
+----+-------+-----+---------------+
|  1 | jason |  18 | 2022-09-27    |
|  2 | tank  |  38 | 2022-09-27    |
+----+-------+-----+---------------+
2 rows in set (0.00 sec)
```

### 2.1.2 删

删除也有两种方式，通过主键删除数据

```python
# 方式一
res = models.User.objects.filter(pk=1).delete()
print(res)

# 方式二
user_obj = models.User.objects.filter(pk=2).first()
user_obj.delete()
```

为什么用`pk`?

```python
"""
pk会自动查找到当前表的主键字段 指代的就是当前表的主键字段
用了pk之后 你就不需要指代当前表的主键字段到底叫什么了
    uid
    pid
    sid
    ...
"""
```
### 2.1.3 改

原表信息

```python
mysql> select * from app01_user;
+----+-------+-----+---------------+
| id | name  | age | register_time |
+----+-------+-----+---------------+
|  5 | jason |  18 | 2022-09-27    |
|  6 | tank  |  38 | 2022-09-27    |
+----+-------+-----+---------------+
2 rows in set (0.00 sec)
```

ORM修改数据表的三种方法：

```python
# 方式一
res = models.User.objects.filter(pk=5).update(name='laosb')

# 方式二
user_obj = models.User.objects.get(pk=6)
user_obj.name = '马有铁'
user_obj.save()

# 方式三
user_obj2 = models.User.objects.filter(pk=5).first()
user_obj2.age = 898
user_obj2.save()
```

>PS:  `get`和`filter`的区别
>
>get方法：
>
>​	返回的直接就是当前数据对象
>
> 	但是该方法不推荐使用
>
>​    一旦数据不存在该方法会直接报错
>
>而filter则不会
>
>​    所以我们还是用filter

查看结果

```mysql
mysql> select * from app01_user;
+----+--------+-----+---------------+
| id | name   | age | register_time |
+----+--------+-----+---------------+
|  5 | laosb  | 898 | 2022-09-27    |
|  6 | 马有铁 |  38 | 2022-09-27    |
+----+--------+-----+---------------+
2 rows in set (0.00 sec)
```

### 2.1.4 查

```python
# 拿到所有数据
res = models.User.objects.all()

# 遍历查询
for i in res:
    print("id: ", i.id, "name:", i.name, "age:", i.age, "register_time: ", i.register_time)
```

输出：

```shell
id:  5 name: laosb age: 898 register_time:  2022-09-27
id:  6 name: 马有铁 age: 38 register_time:  2022-09-27
```

## 2.2 必知必会13条

### 2.2.1 查询所有数据

`all()`方法：

```python
models.User.objects.all()
```

### 2.2.2带有过滤条件的查询

`filter()`方法

```python
# 查询姓名是“马有铁”的数据, 默认是集合序列，需要加first()取出来第一个
res2 = models.User.objects.filter(name="马有铁").first()
print("id: ", res2.id, "name:", res2.name, "age:", res2.age, "register_time: ", res2.register_time)
```

输出：

```python
id:  6 name: 马有铁 age: 38 register_time:  2022-09-27
```

#### 2.2.2.1 双下划线查询
```python
# 1 年龄大于35岁的数据
# res = models.User.objects.filter(age__gt=35)
# print(res)
# 2 年龄小于35岁的数据
# res = models.User.objects.filter(age__lt=35)
# print(res)
# 大于等于 小于等于
# res = models.User.objects.filter(age__gte=32)
# print(res)
# res = models.User.objects.filter(age__lte=32)
# print(res)

# 年龄是18 或者 32 或者40
# res = models.User.objects.filter(age__in=[18,32,40])
# print(res)

# 年龄在18到40岁之间的  首尾都要
# res = models.User.objects.filter(age__range=[18,40])
# print(res)

# 查询出名字里面含有s的数据  模糊查询
# res = models.User.objects.filter(name__contains='s')
# print(res)
#
# 是否区分大小写  查询出名字里面含有p的数据  区分大小写
# res = models.User.objects.filter(name__contains='p')
# print(res)
# 忽略大小写
# res = models.User.objects.filter(name__icontains='p')
# print(res)

# res = models.User.objects.filter(name__startswith='j')
# res1 = models.User.objects.filter(name__endswith='j')
#
# print(res,res1)

# 查询出注册时间是 2020 1月
# res = models.User.objects.filter(register_time__month='1')
# res = models.User.objects.filter(register_time__year='2020')
```

### 2.2.3 直接拿数据对象

`get()`方法

使用此方法，当条件不存在时，将直接报错

```python
# 拿到名称是“马有铁”的数据对象
myt_obj = models.User.objects.filter(name="马有铁").get()
# 输出对象值
print(myt_obj.name)
```

### 2.2.4 拿第一个/最后一个元素

新增一条数据

```python
res = models.User.objects.create(name="张三", age=12, register_time=datetime.datetime.now())
```

`first()`方法拿第一个元素

```python
res3 = models.User.objects.first()
print(res3.id, res3.name)

# 输出
5 laosb
```

`last()`方法拿最有一个元素

```python
res3 = models.User.objects.last()
print(res3.id, res3.name)

# 输出
7 张三
```

### 2.2.5 获取指定的数据字段

`values()`获取name和age字段，**列表套字典**

```python
res4 = models.User.objects.values('name', 'age')
print(res4)

# 输出
<QuerySet [{'name': 'laosb', 'age': 898}, {'name': '马有铁', 'age': 38}, {'name': '张三', 'age': 12}]>
```

`values_list()`获取name和age字段，**列表套元组**

```python
res5 = models.User.objects.values_list('name', 'age')
print(res5)

# 输出
<QuerySet [('laosb', 898), ('马有铁', 38), ('张三', 12)]>
```

### 2.2.6 去重

当前数据如下：

```python
+----+--------+-----+---------------+
| id | name   | age | register_time |
+----+--------+-----+---------------+
|  5 | laosb  | 898 | 2022-09-27    |
|  6 | 马有铁 |  38 | 2022-09-27    |
|  7 | 张三   |  12 | 2022-09-27    |
|  8 | 张三   |  12 | 2022-09-27    |
+----+--------+-----+---------------+
```

没有去重之前查询

```python
res5 = models.User.objects.values_list('name', 'age')

# 输出
<QuerySet [('laosb', 898), ('马有铁', 38), ('张三', 12), ('张三', 12)]>
```

使用`distinct()`去重

```python
res5 = models.User.objects.values_list('name', 'age').distinct()
print(res5)

# 输出
<QuerySet [('laosb', 898), ('马有铁', 38), ('张三', 12)]>
```

### 2.2.7 排序

`order_by()`排序（升）

```python
res6 = models.User.objects.order_by('age')
for i in res6:
    print("id: ", i.id, "name:", i.name, "age:", i.age, "register_time: ", i.register_time)
    
# 输出
id:  7 name: 张三 age: 12 register_time:  2022-09-27
id:  8 name: 张三 age: 12 register_time:  2022-09-27
id:  6 name: 马有铁 age: 38 register_time:  2022-09-27
id:  5 name: laosb age: 898 register_time:  2022-09-27
```

`order_by()`排序（降）

```python
res6 = models.User.objects.order_by('-age')
```

### 2.2.9 反转

`reverse()`反转的前提是数据已经排序过

```python
res6 = models.User.objects.order_by('age').reverse()
for i in res6:
    print("id: ", i.id, "name:", i.name, "age:", i.age, "register_time: ", i.register_time)
    
# 输出，升序已经反转成降序了
id:  5 name: laosb age: 898 register_time:  2022-09-27
id:  6 name: 马有铁 age: 38 register_time:  2022-09-27
id:  7 name: 张三 age: 12 register_time:  2022-09-27
id:  8 name: 张三 age: 12 register_time:  2022-09-27
```

### 2.2.10 统计当前数据的个数

`count()`统计个数

```python
res6 = models.User.objects.order_by('age').reverse().count()
print(res6)

# 输出
4
```

### 2.2.10 排除数据

`exclude()`将name='张三'的数据排除在外

```python
res6 = models.User.objects.exclude(name='张三').all()
for i in res6:
    print("id: ", i.id, "name:", i.name, "age:", i.age, "register_time: ", i.register_time)
```

输出

```python
id:  5 name: laosb age: 898 register_time:  2022-09-27
id:  6 name: 马有铁 age: 38 register_time:  2022-09-27
```

### 2.2.10 查询数据是否存在

`exists()`查询数据是否存在

```python
res6 = models.User.objects.filter(pk=10).exists()
print(res6)
res7 = models.User.objects.filter(pk=6).exists()
print(res6)

# 输出
False
True
```

## 2.3 查看内部sql语句的方式

### 2.3.1 方式一

queryset对象能够点击query查看内部的sql语句

```python
res6 = models.User.objects.values_list('name', 'age')
print(res6.query)

# 输出
SELECT `app01_user`.`name`, `app01_user`.`age` FROM `app01_user`
```

### 2.3.2 方式二

所有的sql语句都能查看，在`settings.py`中添加

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

尝试查询数据的输出

```python
...
(0.000) SELECT `app01_user`.`id`, `app01_user`.`name`, `app01_user`.`age`, `app01_user`.`register_time` FROM `app01_user` ORDER BY `app01_user`.`id` DESC LIMIT 1; args=()
```



# 三、多表操作

## 3.1 前期准备

编辑`models.py`创建ORM对象

```python
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)
    publish_date = models.DateField(auto_now=True)

    # 添加表关系
    # 一对多
    publish = models.ForeignKey(to='Publish', on_delete=models.CASCADE)
    # 多对多
    author = models.ManyToManyField(to='Author')


class Publish(models.Model):
    name = models.CharField(max_length=32)
    addr = models.CharField(max_length=64)
    email = models.EmailField()  # 默认是varchar(254)


class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()

    # 添加表关系：一对一
    author_detail = models.OneToOneField(to='AuthorDetail', on_delete=models.CASCADE)


class AuthorDetail(models.Model):
    phone = models.BigIntegerField()
    addr = models.CharField(max_length=64)
```

同步到数据库

```python
python manage.py makemigrations
python manage.py migrate
```

查看数据库，确定创建成功

```mysql
mysql> show tables;
+----------------------------+
| Tables_in_day64            |
+----------------------------+
| app01_author	# 作者表            
| app01_authordetail	# 作者信息     
| app01_book	# 书表                
| app01_book_author	# 书、作者（虚拟表）        
| app01_publish	# 出版社表              
...
```

## 3.2 一对多外键增删改

### 3.2.1 增

准备出版社数据，案例：一个Publish对应多本Book

```python
mysql> select * from app01_publish;
+----+----------------+--------------+-----------------+
| id | name           | addr         | email           |
+----+----------------+--------------+-----------------+
|  1 | 朝阳群众出版社 | 北京市朝阳区 | 123@qq.com      |
|  2 | 不明所以出版社 | 湖南省长沙市 | 456@163.com     |
|  3 | 一本正经出版社 | 广东省深圳市 | 666@hotmail.com |
|  4 | 没头脑出版社   | 银河系土星   | tx@666.com      |
+----+----------------+--------------+-----------------+
```

增之前先修改`setting.py`取消外键检测

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'day64',
        'USER': 'root',
        'PASSWORD': '123123',
        'HOST': '127.0.0.1',
        'PORT': 3306,
        'CHARSET': 'utf8',
        # 新增
        'OPTIONS': {
            "init_command" : "SET foreign_key_checks = 0;",
        }
    }
}
```

增数据的两种方式

```python
# 方式一，直接写实际字段id
models.Book.objects.create(title='论语',price=899.23,publish_id=1)
models.Book.objects.create(title='聊斋',price=444.23,publish_id=2)
models.Book.objects.create(title='老子',price=333.66,publish_id=1)


# 方式二，使用虚拟字段和对象新增
publish_obj = models.Publish.objects.filter(pk=2).first()	# 这个必须能获取到对象，否则后面会报错
models.Book.objects.create(title='红楼梦', price=666.23, publish=publish_obj)
```

查询数据库，已写入成功

```python
mysql> select * from app01_book;
+----+--------+--------+--------------+------------+
| id | title  | price  | publish_date | publish_id |
+----+--------+--------+--------------+------------+
|  6 | 论语   | 899.23 | 2022-09-27   |          1 |
|  7 | 聊斋   | 444.23 | 2022-09-27   |          2 |
|  8 | 老子   | 333.66 | 2022-09-27   |          1 |
|  9 | 红楼梦 | 666.23 | 2022-09-27   |          2 |
+----+--------+--------+--------------+------------+
5 rows in set (0.00 sec)
```

### 3.2.2 删

删除主键为`1`的出版社

```python
models.Publish.objects.filter(pk=1).delete()
```

查询，因为是`级联更新，级联删除`，删除出版社后，对应的书也删除了

```python
mysql> select * from app01_publish;
+----+----------------+--------------+-----------------+
| id | name           | addr         | email           |
+----+----------------+--------------+-----------------+
|  2 | 不明所以出版社 | 湖南省长沙市 | 456@163.com     |
|  3 | 一本正经出版社 | 广东省深圳市 | 666@hotmail.com |
|  4 | 没头脑出版社   | 银河系土星   | tx@666.com      |
+----+----------------+--------------+-----------------+
3 rows in set (0.00 sec)

mysql> select * from app01_book;
+----+--------+--------+--------------+------------+
| id | title  | price  | publish_date | publish_id |
+----+--------+--------+--------------+------------+
|  7 | 聊斋   | 444.23 | 2022-09-27   |          2 |
|  9 | 红楼梦 | 666.23 | 2022-09-27   |          2 |
+----+--------+--------+--------------+------------+
```

### 3.2.3 改

修改数据的两种方式

```python
# 方式一
models.Book.objects.filter(pk=7).update(publish_id=1)

# 方式二
# 获取出版社对象
publish_obj = models.Publish.objects.filter(pk=3).first()
# 将出版设和书绑定
models.Book.objects.filter(pk=9).update(publish=publish_obj)
```

查询数据库，修改成功

```mysql
mysql> select * from app01_book;
+----+--------+--------+--------------+------------+
| id | title  | price  | publish_date | publish_id |
+----+--------+--------+--------------+------------+
|  7 | 聊斋   | 444.23 | 2022-09-27   |          1 |
|  9 | 红楼梦 | 666.23 | 2022-09-27   |          3 |
+----+--------+--------+--------------+------------+
```

## 3.3 多对多外键增删改查

多对多的案例：一本书有多个作者

添加作者，和作者描述

```mysql
mysql> select * from app01_author;
+----+------+-----+------------------+
| id | name | age | author_detail_id |
+----+------+-----+------------------+
|  1 | 张三 |  11 |                1 |
|  2 | 李四 |  28 |                2 |
|  3 | 王五 |  80 |                3 |
+----+------+-----+------------------+

mysql> select * from app01_authordetail;
+----+--------+------+
| id | phone  | addr |
+----+--------+------+
|  1 | 123456 | 长沙 |
|  2 | 234567 | 深圳 |
|  3 | 345678 | 北京 |
+----+--------+------+
3 rows in set (0.00 sec)
```

### 3.3.1 增

给书籍增加作者，两种方式

```python
# 方式一
book_obj = models.Book.objects.filter(pk=7).first()
print(book_obj.author) # 类似于已经到了第三张表（虚拟的book_author表）
book_obj.author.add(2, 3)    # 绑定主键为2,3的作者

# 方式二
# 获取三个作者的对象
author_obj1 = models.Author.objects.filter(pk=1).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
author_obj3 = models.Author.objects.filter(pk=3).first()
# 将作者关联到书
book_obj.author.add(author_obj1)
```

查询数据，关联成功，虚拟表更新

```mysql
mysql> select * from app01_book_author;
+----+---------+-----------+
| id | book_id | author_id |
+----+---------+-----------+
|  5 |       7 |         1 |
|  1 |       7 |         2 |
|  2 |       7 |         3 |
+----+---------+-----------+
3 rows in set (0.00 sec)
```

### 3.3.2 删

删除的两种方法

```python
# 方法一
book_obj = models.Book.objects.filter(pk=7).first()
book_obj.author.remove(2,3)

# 方法二
author_obj = models.Author.objects.filter(pk=2).first()
author_obj1 = models.Author.objects.filter(pk=3).first()
book_obj.authors.remove(author_obj,author_obj1)
```

查询

```mysql
mysql> select * from app01_book_author;
+----+---------+-----------+
| id | book_id | author_id |
+----+---------+-----------+
|  5 |       7 |         1 |
+----+---------+-----------+
1 row in set (0.00 sec)
```

### 3.3.3 改

```python
# 方法一
book_obj.authors.set([1,2])  # 括号内必须给一个可迭代对象
book_obj.authors.set([3])  # 括号内必须给一个可迭代对象

# 方法二
author_obj = models.Author.objects.filter(pk=2).first()
author_obj1 = models.Author.objects.filter(pk=3).first()
book_obj.authors.set([author_obj,author_obj1])  # 括号内必须给一个可迭代对象
```

查询

```mysql
mysql> select * from app01_book_author;
+----+---------+-----------+
| id | book_id | author_id |
+----+---------+-----------+
|  6 |       7 |         2 |
|  7 |       7 |         3 |
+----+---------+-----------+
```

### 3.3.4 清空

```python
book_obj.authors.clear()
```

## 3.4 正反向的概念

```python
# 正向
外键字段在我手上那么，我查你就是正向
# 反向
外键字段如果不在手上，我查你就是反向
  
book >>>外键字段在书那儿(正向)>>> publish
publish	>>>外键字段在书那儿(反向)>>>book
  
一对一和多对多正反向的判断也是如此
  
"""
正向查询按字段
反向查询按表名小写
				_set
				...
"""

```

# 四、多表（跨表）查询

## 4.1 对象查询（子查询）

>在书写orm语句的时候跟写sql语句一样的
>不要企图一次性将orm语句写完 如果比较复杂 就写一点看一点
>
>正向什么时候需要加.all()
>    当你的结果可能有多个的时候就需要加.all()
>    如果是一个则直接拿到数据对象
>        book_obj.publish
>        book_obj.authors.all()
>        author_obj.author_detail

案例一：查询书籍主键为9的出版社

```python
book_obj = models.Book.objects.filter(pk=9).first()
res = book_obj.publish
print(res.name)
print(res.addr)

# 输出
一本正经出版社
广东省深圳市
```

案例二：查询书籍主键为7的作者

```python
book_obj = models.Book.objects.filter(pk=7).first()
res = book_obj.author.all()
for i in res:
    print(i.name)

# 输出
李四
王五
```

案例三：查询作者"张三"的电话

```python
author_obj = models.Author.objects.filter(name='张三').first()
res = author_obj.author_detail
print(res.phone)

# 输出
123456
```

案例四、查询出版社是“没头脑出版社“的书

```python
# 先给”没头脑出版社“关联两本书
models.Book.objects.create(title='大话西游', price=343.2, publish_id=4)
models.Book.objects.create(title='进化论', price=3222, publish_id=4)

# 查询
publish_obj = models.Publish.objects.filter(name='没头脑出版社').first()
res = publish_obj.book_set.all()	# 反向需要加_set
for i in res:
    print(i.title, i.price)
    
# 输出
大话西游 343.20
进化论 3222.00
```

案例五、查询作者‘李四’写过的书

```python
author_obj = models.Author.objects.filter(name='李四').first()
res = author_obj.book_set.all()
for i in res:
    print(i.title)
    
# 输出
聊斋
```

案例六：查询手机号是'234567'的作者姓名

```python
authordetail_obj = models.AuthorDetail.objects.filter(phone=234567).first()
res = authordetail_obj.author
print(res.name)

# 输出
李四
```

## 4.2 双下划线查询（联表查询）

案例一：查询作者'张三'的手机号和年龄

```python
# 正向
res = models.Author.objects.filter(name='张三').values('author_detail__phone', 'age')
print(res)

# 反向
res2 = models.AuthorDetail.objects.filter(author__name='张三').values('phone', 'author__age')
print(res2)
```

案例二：查询书籍主键为9的出版社名称和书的名称

```python
# 正向
res = models.Book.objects.filter(pk=9).values('publish__name', 'title')
print(res)
# 反向
res2 = models.Publish.objects.filter(book__id=9).values('name', 'book__title')
print(res2)

# 输出
<QuerySet [{'publish__name': '一本正经出版社', 'title': '红楼梦'}]>
<QuerySet [{'name': '一本正经出版社', 'book__title': '红楼梦'}]>
```

案例三：查询书籍主键为7的作者姓名

```python
# 正向
res = models.Book.objects.filter(pk=7).values('author__name')
print(res)
# 反向
res2 = models.Author.objects.filter(book__id=7).values('name')
print(res2)

# 输出
<QuerySet [{'author__name': '李四'}, {'author__name': '王五'}]>
<QuerySet [{'name': '李四'}, {'name': '王五'}]>
```

案例四：查询书籍主键市7的作者的手机号

```python
# 正向
res = models.Book.objects.filter(pk=7).values('author__author_detail__phone')
print(res)
# 反向报错！不能这样写
#res2 = models.AuthorDetail.objects.filter(book__id=7).values('phone')
#print(res2)

# 输出
<QuerySet [{'author__author_detail__phone': 234567}, {'author__author_detail__phone': 345678}]>
```

# 五、周末作业

```python
"""
今日作业
必做题
1.整理今日内容 用自己的话术整理到博客中(切勿直接复制粘贴)
独立完成以下任务
2.自己手动创建图书管理系统表及数据录入  
3.独立完成单表查询N条方法，双下划线方法
4.将课上orm题目摘出来，自己完成orm语句书写，体会orm简便之处
选做题
1.图书管理系统		图书表的增删改查
	（只需要完成图书表的就可以）
"""
```















