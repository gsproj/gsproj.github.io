---
title: Day46-数据库（三）
date: 2022-07-18 10:52:22
categories:
- Python
- 03_MySQL
tags:
---

“Day46-数据库(三)学习笔记”



# 一、约束条件补充

我们已学习了三个约束条件，分别是：null/not null、unsigned、zerofill, 接下来将补充学习4个常用的约束条件。

它们分别是：default、unique、primary key、auto_increment

## 1.1 default(默认值)

```python
# 创建表t1
create table t1(
	id int,
    name char(16)
);

# 补充知识点  插入数据的时候可以指定字段
insert into t1(name,id) values('jason',1);


# default约束条件
create table t2(
	id int,
    name char(16),
    gender enum('male','female','others') default 'male'
);
# 只插入id和name，gender将默认位male
insert into t2(id,name) values(1,'jason');
# 也可以三个字段都插入
insert into t2 values(2,'egon','female');
```

## 1.2 unique(唯一)

```python
# 单列唯一
mysql> create table t3(id int unique, name char(16));
mysql> insert into t3 values(1, '张三');	# 正常
Query OK, 1 row affected (0.01 sec)
mysql> insert into t3 values(1, '李四');	# 报错
ERROR 1062 (23000): Duplicate entry '1' for key 't3.id'

# 联合唯一
"""
ip和port
单个都可以重复 但是加载一起必须是唯一的
"""
mysql> create table t4 (id int, ip char(16),
                        port int,
                        unique(id, port)
                       );
mysql> insert into t4 values(1, '127.0.0.1', 3308); # 正常
mysql> insert into t4 values(2, '127.0.0.1', 3309); # 只同id，正常
mysql> insert into t4 values(2, '168.0.0.2', 3308); # 只同port，正常
mysql> insert into t4 values(2, '127.0.0.1', 3308); # ip和port都相同，报错
ERROR 1062 (23000): Duplicate entry '2-3308' for key 't4.id'
```

## 1.3 primary key(主键)

```python
"""
1.单单从约束效果上来看primary key等价于not null + unique
非空且唯一！！！
"""
create table t5(id int primary key);
insert into t5 values(null);  报错
insert into t5 values(1),(1);  报错
insert into t5 values(1),(2); 

"""
2.它除了有约束效果之外 它还是Innodb存储引擎组织数据的依据
Innodb存储引擎在创建表的时候必须要有primary key
因为它类似于书的目录 能够帮助提示查询效率并且也是建表的依据
"""
# 1 一张表中有且只有一个主键 如果你没有设置主键 那么会从上往下搜索直到遇到一个非空且唯一的字段将它自动升级为主键
create table t6(
	id int,
    name char(16),
    age int not null unique,	# 将自动升级位主键
    addr char(32) not null unique
);

# 2 如果表中没有主键也没有其他任何的非空且唯一字段 那么Innodb会采用自己内部提供的一个隐藏字段作为主键，隐藏意味着你无法使用到它 就无法提示查询速度

# 3 一张表中通常都应该有一个主键字段 并且通常将id/uid/sid字段作为主键
# 单个字段主键
create table t5(
    id int primary key
	name char(16)
);
# 联合主键(多个字段联合起来作为表的主键 本质还是一个主键)
create table t7(
    ip char(16),
    port int,
    primary key(ip,port)
);

"""
也意味着 以后我们在创建表的时候id字段一定要加primary key
"""
```

## 1.4 auto_increment(自增)

```python
# 当编号特别多的时候 人为的去维护太麻烦
create table t8(
	id int primary key auto_increment,
    name char(16)
);
insert into t8(name) values('jason'),('egon'),('kevin');

# 注意auto_increment通常都是加在主键上的，不能给普通字段加，否则会报错
create table t9(
	id int primary key auto_increment,
    name char(16),
    cid int auto_increment
);
ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key	
```

**结论**

```python
"""
以后在创建表的id(数据的唯一标识id、uid、sid)字段的时候
id int primary key auto_increment
"""
```

**补充**

```python
delete from t1 where id=1 删除表中数据后 主键的自增不会停止，可以使用where
truncate t1  清空表所有数据并且重置主键，不能使用where
```



# 二、表与表之间建关系

## 2.1 什么是表关系

当我们有需要，需要定义一张员工表，表中有很多字段

| 编号 | 姓名 | 性别   | 部门名称 | 部门信息 |
| ---- | ---- | ------ | -------- | -------- |
| id   | name | gender | dep_name | dep_desc |

该表有三个问题：

1. 该表的组织结构不是很清晰(可忽视)
2. 浪费硬盘空间(可忽视)
3. 数据的扩展性极差(无法忽视的)

就类似于将所有代码都写在了一个py文件里，有些混乱

如何优化？ 可以将员工表拆分位员工表和部门表，此时就需要使用`外键(foreign key)`将表与表关联到一起

## 2.2 表关系

表与表之间最多只有四种关系

1. 一对多关系（多对一也叫一对多）
2. 多对多关系
3. 一对一
4. 没有关系

接下来分别介绍前三种关系

### 2.2.1 一对多关系

思考：员工表与部门表的关系

一个员工能否对应多个部门？ --- 不能

一个部门能否对应多个员工？ --- 可以

 得出结论：员工表与部门表示单向的一对多，所以表关系就是一对多

>☀️PS：
>
>foreign key定义准则：
>
>1. 一对多表关系   外键字段建在多的一方
>2. 在创建表的时候 一定要先建被关联表
>3. 在录入数据的时候 也必须先录入被关联表

员工表( *employee*)和部门表(*department*)一对多关系的案例：

```mysql
# 创建部门表（先创建被关联表）
mysql> create table dep(
    -> id int primary key auto_increment,
    -> dep_name char(16),
    -> dep_desc char(32)
    -> );

# 创建员工表
mysql> create table emp(
    -> id int primary key auto_increment,
    -> name char(16),
    -> gender enum('male', 'female', 'other') default 'male',
    -> dep_id int,
    -> foreign key(dep_id) references dep(id)	# 员工是多的一方，外键写这
    -> );
    
# 部门表插入数据，成功
mysql> insert into dep(dep_name, dep_desc) values('财务部', '报销效率高半年能下'), ('管理部', '入职流程一条龙'), ('技术部', '蓝翔CV班');
mysql> select * from dep;
+----+----------+--------------------+
| id | dep_name | dep_desc           |
+----+----------+--------------------+
|  1 | 财务部   | 报销效率高半年能下 |
|  2 | 管理部   | 入职流程一条龙     |
|  3 | 技术部   | 蓝翔CV班           |
+----+----------+--------------------+

# 员工表插入数据，成功
mysql> insert into emp(name, gender, dep_id) values('张三','male', 1), ('李四','female',2), ('王五', 'other',1);

mysql> select * from emp;
+----+------+--------+--------+
| id | name | gender | dep_id |
+----+------+--------+--------+
|  1 | 张三 | male   |      1 |
|  2 | 李四 | female |      2 |
|  3 | 王五 | other  |      1 |
+----+------+--------+--------+

# 员工表插入数据，如果外键不存在，将插入失败
mysql> insert into emp(name, gender, dep_id) values('杨六','male', 5);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`day46`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`dep_id`) REFERENCES `dep` (`id`))
```

### 2.2.2 级联更新/级联删除

在2.2.1的案例中，我们可以正常添加数据，但如果需要修改/删除表中的数据，就会显得有点麻烦

```mysql
# 修改dep表中的id字段，失败，已关联外键
mysql> update dep set id=200 where id=2;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`day46`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`dep_id`) REFERENCES `dep` (`id`))

# 删除dep表中的数据，失败，已关联外键
mysql> delete from dep where id=2;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`day46`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`dep_id`) REFERENCES `dep` (`id`))
```

需要先删除对应的员工数据，之后再删除/修改部门数据，操作太过繁琐

需要真正做到数据之间有关系，要做到：

更新就同步更新	----	`级联更新`

删除就同步删除	----	`级联删除`

```mysql
create table emp(
	id int primary key auto_increment,
    name char(16),
    gender enum('male','female','others') default 'male',
    dep_id int,
    foreign key(dep_id) references dep(id) 
    on update cascade  # 同步更新
    on delete cascade  # 同步删除
);

# 再尝试更新， 成功
mysql> update dep set id=200 where id=2;
+----+------+--------+--------+
| id | name | gender | dep_id |
+----+------+--------+--------+
|  1 | 张三 | male   |      1 |
|  2 | 李四 | female |    200 |
|  3 | 王五 | other  |      1 |
+----+------+--------+--------+

# 再尝试删除，成功
mysql> delete from dep where id=1;
+----+------+--------+--------+
| id | name | gender | dep_id |
+----+------+--------+--------+
|  2 | 李四 | female |    200 |
+----+------+--------+--------+
```

### 2.2.3 多对多

以图书表和作者表为例，

一本书可以有多个作者

一个作者也可以有多本书

```python
"""
图书表和作者表
"""
create table book(
	id int primary key auto_increment,
    title varchar(32),
    price int,
    author_id int,
    foreign key(author_id) references author(id) 
    on update cascade  # 同步更新
    on delete cascade  # 同步删除
);
create table author(
	id int primary key auto_increment,
    name varchar(32),
    age int,
    book_id int,
    foreign key(book_id) references book(id) 
    on update cascade  # 同步更新
    on delete cascade  # 同步删除
);
"""
按照上述的方式创建 一个都别想成功！！！
其实我们只是想记录书籍和作者的关系
针对多对多字段表关系 不能在两张原有的表中创建外键
需要你单独再开设一张 专门用来存储两张表数据之间的关系
"""
create table book(
	id int primary key auto_increment,
    title varchar(32),
    price int
);
create table author(
	id int primary key auto_increment,
    name varchar(32),
    age int
);
create table book2author(
	id int primary key auto_increment,
    author_id int,
    book_id int,
    foreign key(author_id) references author(id) 
    on update cascade  # 同步更新
    on delete cascade,  # 同步删除
    foreign key(book_id) references book(id) 
    on update cascade  # 同步更新
    on delete cascade  # 同步删除
);

# 创建数据的顺序
先给book/author表添加数据
再给book2author表添加数据
```

### 2.2.4 一对一

```python
"""
id name age addr phone hobby email........
如果一个表的字段特别多 每次查询又不是所有的字段都能用得到
将表一分为二  
	用户表
		用户表
			id name age
		用户详情表
			id addr phone hobby email........
	
	站在用户表
		一个用户能否对应多个用户详情   不能！！！
	站在详情表
		一个详情能否属于多个用户      不能！！！
	结论:单向的一对多都不成立 那么这个时候两者之间的表关系
		就是一对一
		或者没有关系(好判断)
"""

# 一对一 外键字段建在任意一方都可以 但是推荐你建在查询频率比较高的表中
create table authordetail(
	id int primary key auto_increment,
    phone int,
    addr varchar(64)
);
create table author(
	id int primary key auto_increment,
    name varchar(32),
    age int,
    authordetail_id int unique, # 约束条件`unique`别漏 
    foreign key(authordetail_id) references authordetail(id) 
    on update cascade  # 同步更新
    on delete cascade  # 同步删除
)
```

### 2.2.5 总结

```PYTHON
"""
表关系的建立需要用到foreign key
	一对多
		外键字段建在多的一方
	多对多
		自己开设第三张存储
	一对一
		建在任意一方都可以 但是推荐你建在查询频率较高的表中

判断表之间关系的方式
	换位思考！！！
		员工与部门
		图书与作者
		作者与作者详情
"""
```

# 三、表操作

## 3.1 修改表(了解)

```python
# MySQL对大小写是不敏感的
# 1、修改表名
	alter table 表名 rename 新表名;

# 2、增加字段
	alter table 表名 add 字段名 字段类型(宽度)  约束条件;
	alter table 表名 add 字段名 字段类型(宽度)  约束条件 first;	
	alter table 表名 add 字段名 字段类型(宽度)  约束条件 after 字段名; 

# 3、删除字段
	alter table 表名 drop 字段名;

# 4、修改字段
	alter table 表名 modify 字段名 字段类型(宽度) 约束条件;
	alter table 表名 change 旧字段名 新字段名 字段类型(宽度) 约束条件;
```

## 3.2 复制表(了解)

```python
"""
我们sql语句查询的结果其实也是一张虚拟表
"""
create table 表名 select * from 旧表;  # 不能复制主键 外键 ...
# 案例
create table new_dep2 select * from dep where id > 3;
```



# 四、练习

练习：账号信息表，用户组，主机表，主机组

```python
#用户表
create table user(
id int not null unique auto_increment,
username varchar(20) not null,
password varchar(50) not null,
primary key(username,password)
);

#用户组表
create table usergroup(
id int primary key auto_increment,
groupname varchar(20) not null unique
);

#主机表
create table host(
id int primary key auto_increment,
ip char(15) not null unique default '127.0.0.1'
);

#业务线表
create table business(
id int primary key auto_increment,
business varchar(20) not null unique
);

#建关系：user与usergroup

create table user2usergroup(
id int not null unique auto_increment,
user_id int not null,
group_id int not null,
primary key(user_id,group_id),
foreign key(user_id) references user(id),
foreign key(group_id) references usergroup(id)
);

#建关系：host与business
create table host2business(
id int not null unique auto_increment,
host_id int not null,
business_id int not null,
primary key(host_id,business_id),
foreign key(host_id) references host(id),
foreign key(business_id) references business(id)
);

#建关系：user与host
create table user2host(
id int not null unique auto_increment,
user_id int not null,
host_id int not null,
primary key(user_id,host_id),
foreign key(user_id) references user(id),
foreign key(host_id) references host(id)
);
```

练习:

```python
# 班级表
cid	caption
# 学生表
sid sname gender class_id
# 老师表
tid	tname
# 课程表
cid	cname	teacher_id
# 成绩表
sid	student_id course_id number
```











