---
title: Day44-数据库（一）
date: 2022-07-18 10:39:22
categories:
- Python
- 03_MySQL
tags:
---

“Day44-数据库(一)学习笔记”

# 一 、数据库管理软件的由来

数据库管理软件是怎么来的？

## 1.1 数据的保存问题

数据要想永久保存，都是保存于文件中，毫无疑问，一个文件仅仅只能存在于某一台机器上。

如果我们暂且忽略直接基于文件来存取数据的效率问题，并且假设程序所有的组件都运行在一台机器上，那么用文

件存取数据，并没有问题。

很不幸，这些假设都是你自己意淫出来的，上述假设存在以下几个问题。。。。。。

**1、程序所有的组件就不可能运行在一台机器上**

- 因为这台机器一旦挂掉则意味着整个软件的崩溃，并且程序的执行效率依赖于承载它的硬件，而一台机器机器的性能总归是有限的，受限于目前的硬件水平，就一台机器的性能垂直进行扩展是有极限的。
- 于是我们只能通过水平扩展来增强我们系统的整体性能，这就需要我们将程序的各个组件分布于多台机器去执行。

**2、数据安全问题**

- 根据1的描述，我们将程序的各个组件分布到各台机器，但需知各组件仍然是一个整体，言外之意，所有组件的数据还是要共享的。但每台机器上的组件都只能操作本机的文件，这就导致了数据必然不一致。

- 于是我们想到了将数据与应用程序分离：把文件存放于一台机器，然后将多台机器通过网络去访问这台机器上的文件（用socket实现），即共享这台机器上的文件,共享则意味着竞争，会发生数据不安全，需要加锁处理。。。。

**3、并发**

根据2的描述，我们必须写一个socket服务端来管理这台机器（数据库服务器）上的文件，然后写一个socket客户端，完成如下功能：

1. 远程连接（支持并发）
2. 打开文件
3. 读写（加锁）
4. 关闭文件

## 1.2 由此而生的管理软件：

我们在编写任何程序之前，都需要事先写好基于网络`操作一台主机上数据文件`的程序（socket服务端与客户端程

序），于是有人将此类程序写成一个专门的处理软件，这就是mysql等数据库管理软件的由来，但mysql解决的不

仅仅是数据共享的问题，还有查询效率，安全性等一系列问题，总之，把程序员从数据管理中解脱出来，专注于自

己的程序逻辑的编写。



# 二、数据库概述

## 2.1 什么是数据（Data）

数据：描述事务的符号记录

符号：既可以是数字，也可以是文字、图片，图像、声音、语言等

数据由多种表现形式，它们都可以经过数字化后存入计算机

在计算机中描述一个事物，就需要抽取这一事物的典型特征，组成一条记录，就相当于文件里的一行内容，如

```python
1 egon,male,18,1999,山东,计算机系,2017,oldboy
```

单纯的一条记录并没有任何意义，如果我们按逗号作为分隔，依次定义各个`字段`的意思，相当于定义表的标题

```python
1 name,sex,age,birth,born_addr,major,entrance_time,school #字段
2 egon,male,18,1999,山东,计算机系,2017,gschool #记录
```

这样我们就可以了解egon，性别为男，年龄18岁，出生于1999年，出生地为山东，2017年考入gschool

## 2.2 什么是数据库（DataBase）

数据库：即存放数据的仓库，只不过这个仓库是在计算机存储设备上，而且数据是按一定的格式存放的

## 2.3 什么是数据库管理系统（DBMS）

在了解了Data与DB的概念后，如何科学地组织和存储数据，如何高效获取和维护数据成了关键

这就用到了一个系统软件---数据库管理系统

如MySQL、Oracle、SQLite、Access、MS SQL Server

mysql主要用于大型门户，例如搜狗、新浪等，它主要的优势就是开放源代码，因为开放源代码这个数据库是免费

的，他现在是甲骨文公司的产品。

oracle主要用于银行、铁路、飞机场等。该数据库功能强大，软件费用高。也是甲骨文公司的产品。

sql server是微软公司的产品，主要应用于大中型企业，如联想、方正等。

## 2.4 数据库服务器、数据管理系统、数据库、表与记录的关系（重点理解！！！）

| 描述           | 案例                        | 备注             |
| -------------- | --------------------------- | ---------------- |
| 记录           | 1，刘海龙，324245234，22    | 文件中的一行内容 |
| 表             | student，scholl，class_list | 文件             |
| 数据库         | oldboy_stu                  | 文件夹           |
| 数据库管理系统 | mysql                       | 软件             |
| 数据库服务器   | 一台计算机                  | 服务器           |

图示如下：

![img](https://pic3.zhimg.com/80/v2-bfdd92b482a8d0c473a92e1d2651f3aa_720w.jpg)



# 三、MySQL介绍

## 3.1 什么是MySQL

MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下公司。MySQL 最流行的关

系型数据库管理系统，在 WEB 应用方面MySQL是最好的 RDBMS (Relational Database Management System，关系数

据库管理系统) 应用软件之一。

常用客户端软件

- mysql自带：如mysql命令，mysqldump命令等
- python模块：如pymysql

## 3.2 数据库管理软件分类

```python
#分两大类：
　　关系型：如sqllite，db2，oracle，access，sql server，MySQL，注意：sql语句通用
　　非关系型：mongodb，redis，memcache

#可以简单的理解为：
    关系型数据库需要有表结构
    非关系型数据库是key-value存储的，没有表结构
```

# 四、下载安装

**Linux版本**

```python
#二进制rpm包安装
yum -y install mysql-server mysql
```

源码安装mysql

```python
1.解压tar包
cd /software
tar -xzvf mysql-5.6.21-linux-glibc2.5-x86_64.tar.gz
mv mysql-5.6.21-linux-glibc2.5-x86_64 mysql-5.6.21

2.添加用户与组
groupadd mysql
useradd -r -g mysql mysql
chown -R mysql:mysql mysql-5.6.21

3.安装数据库
su mysql
cd mysql-5.6.21/scripts
./mysql_install_db --user=mysql --basedir=/software/mysql-5.6.21 --datadir=/software/mysql-5.6.21/data

4.配置文件
cd /software/mysql-5.6.21/support-files
cp my-default.cnf /etc/my.cnf
cp mysql.server /etc/init.d/mysql
vim /etc/init.d/mysql   #若mysql的安装目录是/usr/local/mysql,则可省略此步
修改文件中的两个变更值
basedir=/software/mysql-5.6.21
datadir=/software/mysql-5.6.21/data

5.配置环境变量
vim /etc/profile
export MYSQL_HOME="/software/mysql-5.6.21"
export PATH="$PATH:$MYSQL_HOME/bin"
source /etc/profile

6.添加自启动服务
chkconfig --add mysql
chkconfig mysql on

7.启动mysql
service mysql start

8.登录mysql及改密码与配置远程访问
mysqladmin -u root password 'your_password'     #修改root用户密码
mysql -u root -p     #登录mysql,需要输入密码
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your_password' WITH GRANT OPTION;     #允许root用户远程访问
mysql>FLUSH PRIVILEGES;     #刷新权限
```

源码安装mariadb

```python
1. 解压
tar zxvf  mariadb-5.5.31-linux-x86_64.tar.gz   
mv mariadb-5.5.31-linux-x86_64 /usr/local/mysql //必需这样，很多脚本或可执行程序都会直接访问这个目录

2. 权限
groupadd mysql             //增加 mysql 属组 
useradd -g mysql mysql     //增加 mysql 用户 并归于mysql 属组 
chown mysql:mysql -Rf  /usr/local/mysql    // 设置 mysql 目录的用户及用户组归属。 
chmod +x -Rf /usr/local/mysql    //赐予可执行权限 

3. 拷贝配置文件
cp /usr/local/mysql/support-files/my-medium.cnf /etc/my.cnf     //复制默认mysql配置 文件到/etc目录 

4. 初始化
/usr/local/mysql/scripts/mysql_install_db --user=mysql          //初始化数据库 
cp  /usr/local/mysql/support-files/mysql.server    /etc/init.d/mysql    //复制mysql服务程序 到系统目录 
chkconfig  mysql on     //添加mysql 至系统服务并设置为开机启动 
service  mysql  start  //启动mysql

5. 环境变量配置
vim /etc/profile   //编辑profile,将mysql的可执行路径加入系统PATH
export PATH=/usr/local/mysql/bin:$PATH 
source /etc/profile  //使PATH生效。

6. 账号密码
mysqladmin -u root password 'yourpassword' //设定root账号及密码
mysql -u root -p  //使用root用户登录mysql
use mysql  //切换至mysql数据库。
select user,host,password from user; //查看系统权限
drop user ''@'localhost'; //删除不安全的账户
drop user root@'::1';
drop user root@127.0.0.1;
select user,host,password from user; //再次查看系统权限，确保不安全的账户均被删除。
flush privileges;  //刷新权限

7. 一些必要的初始配置
1）修改字符集为UTF8
vi /etc/my.cnf
在[client]下面添加 default-character-set = utf8
在[mysqld]下面添加 character_set_server = utf8
2）增加错误日志
vi /etc/my.cnf
在[mysqld]下面添加：
log-error = /usr/local/mysql/log/error.log
general-log-file = /usr/local/mysql/log/mysql.log
3) 设置为不区分大小写，linux下默认会区分大小写。
vi /etc/my.cnf
在[mysqld]下面添加：
lower_case_table_name=1

修改完重启：#service  mysql  restart
```

**Window版本**

安装

```python
#1、下载：MySQL Community Server 5.7.16
http://dev.mysql.com/downloads/mysql/

#2、解压
如果想要让MySQL安装在指定目录，那么就将解压后的文件夹移动到指定目录，如：C:\mysql-5.7.16-winx64

#3、添加环境变量
【右键计算机】--》【属性】--》【高级系统设置】--》【高级】--》【环境变量】--》【在第二个内容框中找到 变量名为Path 的一行，双击】 --> 【将MySQL的bin目录路径追加到变值值中，用 ； 分割】

#4、初始化
mysqld --initialize-insecure

#5、启动MySQL服务
mysqld # 启动MySQL服务

#6、启动MySQL客户端并连接MySQL服务
mysql -u root -p # 连接MySQL服务器
```

将MySQL服务制作成windows服务

```python
上一步解决了一些问题，但不够彻底，因为在执行【mysqd】启动MySQL服务器时，当前终端会被hang住，那么做一下设置即可解决此问题：



注意：--install前，必须用mysql启动命令的绝对路径
# 制作MySQL的Windows服务，在终端执行此命令：
"c:\mysql-5.7.16-winx64\bin\mysqld" --install

# 移除MySQL的Windows服务，在终端执行此命令：
"c:\mysql-5.7.16-winx64\bin\mysqld" --remove



注册成服务之后，以后再启动和关闭MySQL服务时，仅需执行如下命令：
# 启动MySQL服务
net start mysql

# 关闭MySQL服务
net stop mysql
```

# 五、mysql软件基本管理

## 5.1 启动查看

linux平台下查看

```python
[root@egon ~]# systemctl start mariadb #启动
[root@egon ~]# systemctl enable mariadb #设置开机自启动
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@egon ~]# ps aux |grep mysqld |grep -v grep #查看进程，mysqld_safe为启动mysql的脚本文件，内部调用mysqld命令
mysql     3329  0.0  0.0 113252  1592 ?        Ss   16:19   0:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql     3488  0.0  2.3 839276 90380 ?        Sl   16:19   0:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
[root@egon ~]# netstat -an |grep 3306 #查看端口
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN  
[root@egon ~]# ll -d /var/lib/mysql #权限不对，启动不成功，注意user和group
drwxr-xr-x 5 mysql mysql 4096 Jul 20 16:28 /var/lib/mysql
```

You must reset your password using ALTER USER statement before executing this statement.

```python
安装完mysql 之后，登陆以后，不管运行任何命令，总是提示这个
mac mysql error You must reset your password using ALTER USER statement before executing this statement.
解决方法：
step 1: SET PASSWORD = PASSWORD('your new password');
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;
```

## 5.2 登录，设置密码

```python
初始状态下，管理员root，密码为空，默认只允许从本机登录localhost
设置密码
[root@egon ~]# mysqladmin -uroot password "123"        设置初始密码 由于原密码为空，因此-p可以不用
[root@egon ~]# mysqladmin -uroot -p"123" password "456"        修改mysql密码,因为已经有密码了，所以必须输入原密码才能设置新密码

命令格式:
[root@egon ~]# mysql -h172.31.0.2 -uroot -p456
[root@egon ~]# mysql -uroot -p
[root@egon ~]# mysql                    以root用户登录本机，密码为空
```

## 5.3 忘记密码

**linux平台下，破解密码的两种方式**

方法一：删除授权库mysql，重新初始化

```python
[root@egon ~]# rm -rf /var/lib/mysql/mysql #所有授权信息全部丢失！！！
[root@egon ~]# systemctl restart mariadb
[root@egon ~]# mysql
```

方法二：启动时，跳过授权库

```python
[root@egon ~]# vim /etc/my.cnf    #mysql主配置文件
[mysqld]
skip-grant-table
[root@egon ~]# systemctl restart mariadb
[root@egon ~]# mysql
MariaDB [(none)]> update mysql.user set password=password("123") where user="root" and host="localhost";
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> \q
[root@egon ~]# #打开/etc/my.cnf去掉skip-grant-table,然后重启
[root@egon ~]# systemctl restart mariadb
[root@egon ~]# mysql -u root -p123 #以新密码登录
```

**windows平台下，5.7版本mysql，破解密码的两种方式：**

方式一

```python
#1 关闭mysql
#2 在cmd中执行：mysqld --skip-grant-tables
#3 在cmd中执行：mysql
#4 执行如下sql：
update mysql.user set authentication_string=password('') where user = 'root';
flush privileges;

#5 tskill mysqld #或taskkill -f /PID 7832
#6 重新启动mysql
```

方式二

```python
#1. 关闭mysql，可以用tskill mysqld将其杀死
#2. 在解压目录下，新建mysql配置文件my.ini
#3. my.ini内容,指定
[mysqld]
skip-grant-tables

#4.启动mysqld
#5.在cmd里直接输入mysql登录，然后操作
update mysql.user set authentication_string=password('') where user='root and host='localhost';

flush privileges;

#6.注释my.ini中的skip-grant-tables，然后启动myqsld，然后就可以以新密码登录了
```

## 5.4 在windows下，为mysql服务指定配置文件

**强调：配置文件中的注释可以有中文，但是配置项中不能出现中文**

my.ini

```python
#在mysql的解压目录下，新建my.ini,然后配置
#1. 在执行mysqld命令时，下列配置会生效，即mysql服务启动时生效
[mysqld]
;skip-grant-tables
port=3306
character_set_server=utf8
default-storage-engine=innodb
innodb_file_per_table=1


#解压的目录
basedir=E:\mysql-5.7.19-winx64
#data目录
datadir=E:\my_data #在mysqld --initialize时，就会将初始数据存入此处指定的目录，在初始化之后，启动mysql时，就会去这个目录里找数据



#2. 针对客户端命令的全局配置，当mysql客户端命令执行时，下列配置生效
[client]
port=3306
default-character-set=utf8
user=root
password=123

#3. 只针对mysql这个客户端的配置，2中的是全局配置，而此处的则是只针对mysql这个命令的局部配置
[mysql]
;port=3306
;default-character-set=utf8
user=egon
password=4573


#！！！如果没有[mysql],则用户在执行mysql命令时的配置以[client]为准
```

## 5.5 统一字符编码

```python
#1. 修改配置文件
[mysqld]
default-character-set=utf8 
[client]
default-character-set=utf8 
[mysql]
default-character-set=utf8

#mysql5.5以上：修改方式有所改动
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8

#2. 重启服务
#3. 查看修改结果：
\s
show variables like '%char%'
```

# 六、初识SQL语句

## 6.1 SQL语句分类

SQL语言主要用于存取数据、查询数据、更新数据和管理关系数据库系统,SQL语言由IBM开发。SQL语言分为3种类型：

1. DDL语句    数据库定义语言： 数据库、表、视图、索引、存储过程，例如CREATE DROP ALTER
2. DML语句    数据库操纵语言： 插入数据INSERT、删除数据DELETE、更新数据UPDATE、查询数据SELECT
3. DCL语句    数据库控制语言： 例如控制用户的访问权限GRANT、REVOKE

## 6.2 基本SQL语句

```python
#1. 操作库
        增：create database db1 charset utf8;
        查：show databases; # 查所有
            show create database db1; # 查单个
        改：alter database db1 charset latin1;
        删: drop database db1;


#2. 操作表
    先切换到文件夹下：use db1
        增：create table t1(id int,name char);
        查：show tables;
        改：alter table t1 modify name char(3); # 字段改属性
        	alter table t8 change id id tinyint unsigned; # 字段改属性和约束条件
            alter table t1 change name name1 char(2); # 字段改名
            alter table t1 add(age tinyint)	# 添加字段
            alter table t8 drop column `age`; # 删除字段
        删：drop table t1;


#3. 操作记录
        增：insert into t1 values(1,'egon1'),(2,'egon2'),(3,'egon3');
        查：select * from t1;
        改：update t1 set name='sb' where id=2;
        删：delete from t1 where id=1;

        清空表：
            delete from t1; #如果有自增id，新增的数据，仍然是以删除前的最后一样作为起始。
            truncate table t1;数据量大，删除速度比上一条快，且直接从零开始，

            auto_increment 表示：自增
            primary key 表示：约束（不能重复且不能为空）；加速查找
```

