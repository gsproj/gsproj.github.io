---
title: 运维之综合架构--07--Nginx(四)LNMP架构拆分
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、拆分数据库	

> 为什么要拆分数据库?
>
> mysql内存占用大，容易引起网页访问速度变慢，甚至oom(out of memory)被系统自动kill掉，不安全

### 1.2 环境准备

| 主机名称 | 应用环境  | 外网地址 | 内网地址    |
| -------- | --------- | -------- | ----------- |
| web01    | nginx+php | 10.0.0.7 | 172.16.1.7  |
| db01     | mysql     |          | 172.16.1.51 |

### 1.2 拆分过程

1、备份172.16.1.7服务器上mysql的数据

```shell
[root@web01 ~]# mysqldump -uroot -p'Bgx123.com' --all-databases --single-transaction > mysql-all.sql
```

2、传输172.16.1.7的备份数据至172.16.1.51的服务器上

```shell
[root@web01 ~]# scp mysql-all.sql root@172.16.1.51:/tmp
```

3、需要先在172.16.1.51服务器上安装mysql服务，然后使用mysql命令进行还原。

```shell
[root@db01 ~]# yum install mariadb-server mariadb -y
[root@db01 ~]# systemctl enable mariadb
[root@db01 ~]# systemctl start mariadb
[root@db01 ~]# mysql </tmp/mysql-all.sql
[root@db01 ~]# systemctl restart mariadb
[root@db01 ~]# mysql -uroot -pBgx123.com
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| edusoho            |
| mysql              |
| performance_schema |
| test               |
| wordpress          |
| zh                 |
+--------------------+
7 rows in set (0.00 sec)
```

4、将web程序连接的本地数据库修改到远程数据库上。

```shell
1）先在本地172.16.1.7服务器上停止本地的数据库
[root@web01 ~]# systemctl disable mariadb
[root@web01 ~]# systemctl stop mariadb

2）在172.16.1.51的服务器上授权远程主机能够能连接mysql数据库
[root@db01 ~]# mysql -uroot -pBgx123.com
MariaDB [(none)]> grant all privileges on *.* to oldboy@'%' identified by 'Bgx123.com';
解释：
*.*: 所有数据库下的所有表
oldboy@'%': 允许所有网段的oldboy用户访问
identify: 设置密码

3）在172.16.1.7服务器上测试远程账户能否连接172.16.1.51的数据库
[root@web01 wordpress]# yum install mariadb -y
[root@web01 wordpress]# mysql -h 172.16.1.51 -uoldboy -pBgx123.com
MariaDB [(none)]> 

4）在172.16.1.7服务器上修改web程序连接数据库的配置文件
[root@web01 wordpress]# vim /code/wordpress/wp-config.php
// ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
/** WordPress数据库的名称 */
define('DB_NAME', 'wordpress');

/** MySQL数据库用户名 */
define('DB_USER', 'oldboy');

/** MySQL数据库密码 */
define('DB_PASSWORD', 'Bgx123.com');

/** MySQL主机 */
define('DB_HOST', '172.16.1.51');
```

5、拆分172.16.1.7wecenter连接远程172.16.1.51数据库信息

```shell
	[root@web01 zh]# grep -R "Bgx123.com" *
		system/config/database.php:  'password' => 'Bgx123.com',
	[root@web01 zh]# vim /code/zh/system/config/database.php 
		$config['driver'] = 'MySQLi';^M
		$config['master'] = array (
		  'charset' => 'utf8',
		  'host' => '172.16.1.51',
		  'username' => 'oldboy',
		  'password' => 'Bgx123.com',
		  'dbname' => 'zh',
		);^M
```

6、拆分172.16.1.7 edusoho连接远程172.16.1.51数据库信息

```shell
[root@web01 edusoho]# vim /code/edusoho/app/config/parameters.yml
database_driver: pdo_mysql
database_host: 172.16.1.51
database_port: 3306
database_name: edusoho
database_user: oldboy
database_password: 'Bgx123.com'

必须清理缓存
[root@web01 edusoho]# rm -rf /code/edusoho/app/cache/*
```

## 二、扩展多台web服务器

```shell
1.统一环境
    0）准备对应的www用户
    [root@web02 ~]# groupadd -g666 www
    [root@web02 ~]# useradd -u666 -g666 www

    1）拷贝web01上面的yum仓库
    [root@web02 ~]# scp root@172.16.1.7:/etc/yum.repos.d/*.repo /etc/yum.repos.d/

    2）安装nginx和php
    [root@web02 ~]# yum -y install nginx php71w php71w-cli php71w-common php71w-devel php71w-embedded php71w-gd php71w-mcrypt php71w-mbstring php71w-pdo php71w-xml php71w-fpm php71w-mysqlnd php71w-opcache php71w-pecl-memcached php71w-pecl-redis php71w-pecl-mongodb

2.统一配置（同步web01上面的配置到web02）
    1）同步nginx
    [root@web02 ~]# rsync  -avz --delete root@172.16.1.7:/etc/nginx/ /etc/nginx/
    [root@web02 ~]# nginx -t
    [root@web02 ~]# systemctl enable nginx
    [root@web02 ~]# systemctl start nginx

    2）同步php（/etc/php-fpm.conf /etc/php-fpm.d  /etc/php.ini）
    [root@web02 ~]# rsync  -avz --delete root@172.16.1.7:/etc/php* /etc/
    [root@web02 ~]# systemctl enable php-fpm
    [root@web02 ~]# systemctl start php-fpm

3.统一代码
    [root@web01 ~]# tar czf code.tar.gz /code				#在web01上打包站点
    [root@web01 ~]# scp code.tar.gz root@172.16.1.8:/tmp	#在web01上将打包好的代码发送给web02
    [root@web02 ~]# tar xf /tmp/code.tar.gz -C /			#在web02上进行解压，并解压到/目录下

4.配置解析，进行访问
```

## 三、NFS共享多台web的静态资源

```shell
1.准备172.16.1.31共享存储服务器，规划目录，配置好权限
    0）创建用户
    [root@nfs ~]# groupadd -g666 www
    [root@nfs ~]# useradd -u666 -g666 www	

    1）安装
    [root@nfs ~]# yum install nfs-utils -y

    2）配置
    [root@nfs ~]# cat /etc/exports
    /data/blog 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
    /data/zh 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
    /data/edu 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)

    3）根据配置，创建目录，准备用户，授权等等
    [root@nfs ~]# rm -rf /data/
    [root@nfs ~]# mkdir /data/{blog,zh,edu} -p
    [root@nfs ~]# chown -R www.www /data/

    4）启动
    [root@nfs ~]# systemctl enable nfs-utils 
    [root@nfs ~]# systemctl restart nfs-utils

2.将图片较多的web02服务器，推送到nfs共享存储上
    http://blog.oldboy.com/wp-content/uploads/2019/01/timg.jpg

    [root@web02 ~]# cd /code/wordpress/wp-content
    [root@web02 wp-content]# scp -r uploads/* root@172.16.1.31:/data/blog/

    注意：需要上nfs服务器上进行重新的递归授权，否则会出现无法上传文件的错误
    [root@nfs ~]# chown -R www.www /data/

3.web01和web02分别都进行挂载，此时图片进行实现了共享
    mount -t nfs 172.16.1.31:/data/blog  /code/wordpress/wp-content/uploads/
```

