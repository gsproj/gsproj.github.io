---
title: Day45-数据库（二）
date: 2022-07-18 10:50:22
categories:
- Python
- 03_MySQL
tags:
---

“Day45-数据库(二)学习笔记”

# 一、存储引擎

日常生活中文件格式有很多中，并且针对不同的文件格式会有对应不同存储方式和处理机制(比如txt、pdf、word、

mp4等等)

针对不同的数据应该有对应的不同的处理机制来存储

`存储引擎`就是`不同的处理机制`

## 1.1 MySQL主要存储引擎

* Innodb

  是MySQL5.5版本及之后默认的存储引擎

  存储数据更加的安全

* myisam

  是MySQL5.5版本之前默认的存储引擎

  速度要比Innodb更快 但是我们更加注重的是数据的安全

* memory

  内存引擎(数据全部存放在内存中) 断电数据丢失

* blackhole

  无论存什么，都立刻消失(黑洞)

## 1.2 查看和设置存储引擎：

1、查看所有存储引擎

```mysql
mysql> show engines;

# 输出
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+---------------------------------------------------------------
....
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
.....
```

2、设置存储引擎

```python
# 设置存储引擎
create table t1(id int) engine=innodb;
create table t2(id int) engine=myisam;
create table t3(id int) engine=blackhole;
create table t4(id int) engine=memory;

# 存数据后的表现
insert into t1 values(1);
insert into t2 values(1);
insert into t3 values(1);	# 存进去后查不到数据：Empty set
insert into t4 values(1);	# 服务端重启后查不到数据，没重启之前可以（模拟断电）
```

# 二、创建表的完整语法

## 2.1 创建表的语法

创建表的基本语法为：

```python
create table 表名(
	字段名1 类型(宽度) 约束条件,
    字段名2 类型(宽度) 约束条件,
    字段名3 类型(宽度) 约束条件
);
```

案例：

```mysql
create table t5(id int, 
                name char(12), 
                age int);
```

## 2.2 创建表的注意事项

1、在同一张表中字段名不能重复

```mysql
mysql> create table t6(id int, name char(12), id int);
ERROR 1060 (42S21): Duplicate column name 'id'
```

2、宽度和约束条件是可选的(可写可不写) 而`字段名和字段类型是必须`的，约束条件写的话 也支持写多个

```python
字段名1 类型(宽度) 约束条件1 约束条件2...,
```

少些了会报错，例如：

```mysql
mysql> create table t7(id);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ')' at line 1
```

3、最后一行不能有逗号，否则会报错：

```mysql
mysql> create table t6(id int, 
                       name char(12), 
                       age int,);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ')' at line 1
```

## 2.3 宽度和约束条件

### 2.3.1 宽度

宽度：一般情况下指的是对存储数据的限制

比如：

```mysql
# 创建表
create table t7(name char);  默认宽度是1

# 插入数据'jason'
insert into t7 values('jason');
# 报错
ERROR 1406 (22001): Data too long for column 'name' at row 1

# 插入数据NULL
insert into t7 values(null);  关键字NULL
Query OK, 1 row affected (0.00 sec)

# 插入数据'j'
insert into t7 values('j');
Query OK, 1 row affected (0.00 sec)
```

关于宽度的限制，跟`严格模式`有关，

5.6版本默认没有开启严格模式 规定只能存一个字符你给了多个字符，那么我会自动帮你截取

5.7版本及以上或者开启了严格模式 那么规定只能存几个，就不能超，

一旦超出范围立刻报错 Data too long for ....

### 2.3.2 约束条件

约束条件：是指在宽度的基础之上增加的额外的约束

比如限制是可以插入空值的约束条件

```python
null：可以插入NULL
not null：不能插入NULL
```

案例：

```python
# 创建表
create table t6(id int null, name char(10) not null, age int);

# 插入数据01，正常
mysql> insert into t6 values(1, '大蒜', 10);
Query OK, 1 row affected (0.00 sec)

# 插入数据02，正常
mysql> insert into t6 values(null, '小葱', 10);
Query OK, 1 row affected (0.00 sec)

# 插入数据03，报错
mysql> insert into t6 values(null, NULL, 10);
ERROR 1048 (23000): Column 'name' cannot be null
```



# 三、基本数据类型

## 3.1 整型

### 3.1.1 整型分类

| 类型      | 无符号范围                                  | 有符号范围                |
| --------- | ------------------------------------------- | ------------------------- |
| TINYINT   | -128 到 127                                 | 0 到 255                  |
| SMALLINT  | -32768 到 32768                             | 0 到 65535                |
| MEDUIMINT | -8388608 到 8388607                         | 0 到 16777215             |
| INT       | -2147483648 到 2147483647                   | 0 到 4294967295           |
| BIGINT    | -9223372036854775808 到 9223372036854775807 | 0 到 18446744073709551615 |

### 3.1.2 无符号和有符号

以TINYINT为例，有两个疑问：

是否有符号？ --- 整型默认情况下都是带符号的

超出会如何？ --- 超出限制只存最大可接受值

来验证这两个问题：

**有符号插入：**

```mysql
# 创建表
create table t8(id tinyint, age tinyint);

# 插入数据，过大会报错
mysql> insert into t8 values(-129, 256);
ERROR 1264 (22003): Out of range value for column 'id' at row 1

# 正常范围可以插入
mysql> insert into t8 values(-128, 127);
Query OK, 1 row affected (0.00 sec)
```

**无符号插入**

```mysql
# 修改为有符号
mysql> alter table t8 change age age tinyint unsigned;
mysql> alter table t8 change id id tinyint unsigned;

# 插入数据，成功
mysql> insert into t8 values(0, 255);
Query OK, 1 row affected (0.00 sec)

# 插入数据，失败，超出范围
mysql> insert into t8 values(-127, 128);
ERROR 1264 (22003): Out of range value for column 'id' at row 1
```

>☀️补充：
>
>整型也可以使用括号指定宽度，这个宽度到底是干嘛的？
>
>```mysql
># 创建表，并指定整型宽度
>create table t12(id int(8));
>insert into t12 values(123456789);
>
># 即使超过了8位，也能成功
>mysql> insert into t9 values(123456789);
>Query OK, 1 row affected (0.00 sec)
>```
>
>这是因为：只有整型括号里面的数字不是表示限制位数
>id int(8)
>	如果数字没有超出8位 那么默认用空格填充至8位
>	如果数字超出了8位 那么有几位就存几位(但是还是要遵守最大范围)
>
>可以在创建时设置`zerofill`,用0填充至8位
>
>```mysql
>mysql> create table t10(id int(8) unsigned zerofill);
>Query OK, 0 rows affected, 2 warnings (0.01 sec)
>
>mysql> insert into t10 values(1);
>Query OK, 1 row affected (0.00 sec)
>
>mysql> select * from t10;
>+----------+
>| id       |
>+----------+
>| 00000001 |
>+----------+
>1 row in set (0.00 sec)
>```

针对整型字段 括号内无需指定宽度 因为它默认的宽度以及足够显示所有的数据了

## 3.2 严格模式

```python
# 1、如何查看严格模式
show variables like "%mode";
或者
select @@sql_mode

模糊匹配/查询
	关键字 like
		%:匹配任意多个字符
        _:匹配任意单个字符

# 2、修改严格模式
	set session  只在当前窗口有效
    set global   全局有效
    
    # 临时关闭严格模式（具体内容依据#1命令得到的结果为准，去除ONLY_FULL_GROUP_BY即可）
    set global sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
    
	修改完之后 重新进入服务端即可
```

## 3.3 浮点型

* 分类

  FLOAT、DOUBLE、DECIMAL

* 作用

  身高、体重、薪资

  ```python
  # 存储限制
  float(255,30)  # 总共255位 小数部分占30位
  double(255,30)  # 总共255位 小数部分占30位
  decimal(65,30)  # 总共65位 小数部分占30位
  
  # 精确度验证
  create table t15(id float(255,30));
  create table t16(id double(255,30));
  create table t17(id decimal(65,30));
  """你们在前期不要给我用反向键 所有的命令全部手敲！！！增加熟练度"""
  
  insert into t15 values(1.111111111111111111111111111111);
  insert into t16 values(1.111111111111111111111111111111);
  insert into t17 values(1.111111111111111111111111111111);
  
  float < double < decimal
  # 要结合实际应用场景 三者都能使用
  ```

## 3.4 字符类型

* 分类

  ```python
  """
  char
  	定长
  	char(4)	 数据超过四个字符直接报错 不够四个字符空格补全
  varchar
  	变长
  	varchar(4)  数据超过四个字符直接报错 不够有几个存几个
  """
  create table t18(name char(4));
  create table t19(name varchar(4));
  
  insert into t18 values('a');
  insert into t19 values('a');
  
  # 介绍一个小方法 char_length统计字段长度
  select char_length(name) from t18;
  select char_length(name) from t19;
  """
  首先可以肯定的是 char硬盘上存的绝对是真正的数据 带有空格的
  但是在显示的时候MySQL会自动将多余的空格剔除
  """
  
  # 再次修改sql_mode 让MySQL不要做自动剔除操作
  set global sql_mode = 'STRICT_TRANS_TABLES,PAD_CHAR_TO_FULL_LENGTH';
  ```

  #### char与varchar对比

  ```python
  """
  char
  	缺点:浪费空间
  	优点:存取都很简单
  		直接按照固定的字符存取数据即可
  		jason egon alex wusir tank 
  		存按照五个字符存 取也直接按照五个字符取
  		
  varchar
  	优点:节省空间
  	缺点:存取较为麻烦
  		1bytes+jason 1bytes+egon 1bytes+alex 1bytes+tank 
  		
  		存的时候需要制作报头
  		取的时候也需要先读取报头 之后才能读取真实数据
  		
  以前基本上都是用的char 其实现在用varchar的也挺多
  """
  
  补充:
      进来公司之后你完全不需要考虑字段类型和字段名
      因为产品经理给你发的邮件上已经全部指明了
  ```

## 3.5 时间类型

* 分类

  | 类型     | 案例                            |
| -------- | ------------------------------- |
  | date     | 年月日：2020-5-4                |
| datetime | 年月日时分秒：2020-5-4 11:11:11 |
  | time     | 时分秒：11:11:11                |
| year     | 年：2020                        |
  

案例如下：

  ```python
  create table student(
  	id int,
      name varchar(16),
      born_year year,
      birth date,
      study_time time,
      reg_time datetime
  );
  insert into student values(1,'egon','1880','1880-11-11','11:11:11','2020-11-11 11:11:11');
  ```

## 3.6 枚举与集合类型

* 分类

  ```python
  """
  枚举(enum)  多选一
  集合(set)   多选多
  """
  ```

* 具体使用

  ```python
  # 1、使用枚举
  create table user(
  	id int,
      name char(16),
      gender enum('male','female','others')
  );
  # 插入数据，正常
  insert into user values(1,'jason','male');  
  # 插入数据，报错
  insert into user values(2,'egon','xxxxooo'); 
  # 枚举字段 后期在存数据的时候只能从枚举里面选择一个存储 
  
  
  # 2、使用集合
  create table teacher(
  	id int,
      name char(16),
      gender enum('male','female','others'),
      hobby set('read','DBJ','hecha')
  );
  insert into teacher values(1,'jason','male','read');  # 正常
insert into teacher values(2,'egon','female','DBJ,hecha');  # 正常
  insert into teacher values(3,'tank','others','生蚝'); # 报错
  ```
# 集合可以只写一个  但是不能写没有列举的
  ```
  












  ```