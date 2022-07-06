---
title: 运维之综合架构--07--Nginx(三)LNMP介绍
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、LNMP简介(需补充)

### 1.1 什么是LNMP

### 1.2 LNMP架构是如何工作的

浏览器 --http--> Nginx(fastcgi_pass) --fastcgi-->php(fastcgi_fpm调动wrapper再调动php解析再调用mysql)

大致流程：

用户在浏览器发起请求，如果请求的是静态资源，Nginx则直接返回，如果请求的是动态资源，Nginx会通过fastcgi协议，将请求交给PHP服务器，再返回动态资源。

### 1.3 LNMP和LAMP的区别是什么

nginx 是以fastcgi协议调用的php
apache是以模块的方式加载的php

## 二、LNMP架构简单搭建

1、准备一台名为nginx的服务器

2、使用官方仓库安装nginx

```shell
# 添加安装源
[root@nginx ~]# cat /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1

#安装Nginx
[root@nginx ~]# yum install nginx -y
```

3、安装php7.1

```shell
[root@nginx ~]# yum remove php-mysql-5.4 php php-fpm php-common # 卸载默认5.4版本的php
[root@nginx ~]# cat /etc/yum.repos.d/php.repo
[php]
name = php Repository
baseurl = http://us-east.repo.webtatic.com/yum/el7/x86_64/
gpgcheck = 0

[root@nginx ~]# yum -y install php71w php71w-cli php71w-common php71w-devel php71w-embedded php71w-gd php71w-mcrypt php71w-mbstring php71w-pdo php71w-xml php71w-fpm php71w-mysqlnd php71w-opcache php71w-pecl-memcached php71w-pecl-redis php71w-pecl-mongodb
```

4、安装maria数据库

```shell
[root@nginx ~]# yum install mariadb-server mariadb -y
```

5、配置nginx和php集成

```shell
[root@web01 conf.d]# cd /etc/nginx/conf.d/
[root@web01 conf.d]# cat php.conf 
server {
	listen 80;
	server_name php.oldboy.com;
	root /code;

	location / {
		index index.php index.html;
	}

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```

6、重载nginx

```shell
[root@web01 conf.d]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web01 conf.d]# systemctl restart nginx
```

7、启动php-fpm，并加入开机自启

```shell
[root@web01 conf.d]# systemctl start php-fpm
[root@web01 conf.d]# systemctl enable  php-fpm
```

8、准备一个php文件，测试nginx和php是否继承成功

```shell
[root@web01 conf.d]# cat /code/page.php
<?php
	phpinfo();
?>
# 测试：网页访问:http://php.oldboy.com/page.php
```

9、启动数据库

```shell
[root@web01 conf.d]# systemctl start mariadb
[root@web01 conf.d]# systemctl enable mariadb
[root@web01 conf.d]# mysqladmin password 'Bgx123.com'		#配置密码（默认mysql是空密码）
[root@web01 conf.d]# mysql -uroot -pBgx123.com				#使用账号和密码登录mysql
```

10、准备一个php文件，测试是否可以正常连接数据库

```she
  <?php
    $servername = "localhost";
    $username = "root";
    $password = "Bgx123.com";

    // 创建连接
    $conn = mysqli_connect($servername, $username, $password);

    // 检测连接
    if (!$conn) {
        die("Connection failed: " . mysqli_connect_error()); // 注意格式
    }
    echo "php连接MySQL数据库成功";
    ?>
```

## 三、案例-搭建wordpress博客

### 3.1 环境准备

| 用途        | 公网IP地址 | 内网IP地址 |
| ----------- | ---------- | ---------- |
| web服务器01 | 10.0.0.7   | 172.16.1.7 |

### 3.2 部署安装

1、添加nginx配置文件

```shell
[root@web01 conf.d]# cat blog.oldboy.com.conf 
server {
	listen 80;
	server_name blog.oldboy.com;
	root /code/wordpress;

	location / {
		index index.php index.html;
	}

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```

2、根据nginx中定义的内容，创建站点目录并进行授权

```shell
[root@web01 conf.d]# mkdir /code
[root@web01 conf.d]# cd /code
[root@web01 code]# wget https://cn.wordpress.org/wordpress-5.0.3-zh_CN.tar.gz
[root@web01 code]# tar xf wordpress-5.0.3-zh_CN.tar.gz
```

3、修改nginx与php-fpm的运行用户为www，并授权代码属主和属组都为www

```shell
#注意：如果没有该用户，启动一定会报错
[root@web01 code]# groupadd -g 666 www
[root@web01 code]# useradd -u666 -g666 www

修改nginx与php-fpm管理进程，的运行身份为www
[root@web01 code]# sed -i '/^user /c user  www;' /etc/nginx/nginx.conf
[root@web01 code]# sed -i '/^user/c user = www' /etc/php-fpm.d/www.conf 
[root@web01 code]# sed -i '/^group/c group = www' /etc/php-fpm.d/www.conf 

一定要重启才生效
[root@web01 code]# systemctl restart nginx
[root@web01 code]# systemctl restart php-fpm

最后授权代码为www
[root@web01 code]# chown -R www.www /code/wordpress	
```

4、创建数据库

```shell
MariaDB [(none)]> create database wordpress;			#创建一个库，名称叫wordpress
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;						#查询该台数据库服务有多少个库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)
```

5、解决nginx上传文件大小限制（默认1M，超过大小会报413）

```shell
在wordpress的nginx配置文件中添加：client_max_body_size 100m;
[root@web01 code]# cat /etc/nginx/conf.d/blog.oldboy.com.conf 
server {
        listen 80;
        server_name blog.oldboy.com;
        root /code/wordpress;
        client_max_body_size 100m;  # 默认1M，超过大小会报413

        location / {
                index index.php index.html;
        }

        location ~ \.php$ {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
重启nginx：systemctl restart nginx	
```

>测试在wordpress页面上传主题或者写文章上传图片，均出现500报错
>查看日志文件/var/log/nginx/err.log
>2021/08/16 18:59:31 [crit] 1228#1228: *2 open() "/var/lib/nginx/tmp/client_body/0000000001" failed (13: Permission denied), client: 10.0.0.1, server: php.gs.com, request: "POST /wp-admin/update.php?action=upload-theme HTTP/1.1", host: "php.gs.com", referrer: "http://php.gs.com/wp-admin/theme-install.php?browse=popular"
>解决方法，参考https://blog.csdn.net/qq_15941409/article/details/114640122
>chown www:www -R /var/lib/nginx/

## 四、案例-搭建wecenter知乎

### 4.1 部署安装

1、添加nginx配置文件

```shell
server {
	listen 80;
	server_name zh.oldboy.com;
	root /code/zh;
	client_max_body_size 100m;

	location / {
		index index.php index.html;
	}

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
[root@web01 code]# systemctl restart nginx
```

2、wencenter部署

```shell
[root@web01 code]# rz -E WeCenter_3-2-1.zip
[root@web01 code]# unzip WeCenter_3-2-1.zip
[root@web01 code]# mv WeCenter_3-2-1 zh
[root@web01 code]# chown -R www.www zh/
```

3、配置数据库

```shell
[root@web01 code]# mysql -uroot -pBgx123.com
MariaDB [(none)]> create database zh;
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| wordpress          |
| zh                 |
+--------------------+
6 rows in set (0.00 sec)
```

## 五、案例-搭建edusoho在线视频教育

### 5.1 部署安装

1、添加nginx配置文件

```shell
[root@web01 code]# cat /etc/nginx/conf.d/edu.oldboy.com.conf 
server {
    listen 80;
    server_name edu.oldboy.com;
    root /code/edusoho/web;
    client_max_body_size 200m;

    location / {
        index app.php;
        try_files $uri @rewriteapp;
    }
    location @rewriteapp {
        rewrite ^(.*)$ /app.php/$1 last;
    }

    location ~ ^/udisk {
        internal;
        root /code/edusoho/app/data/;
    }

    location ~ ^/(app|app_dev)\.php(/|$) {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        fastcgi_param  HTTPS              off;
        fastcgi_param HTTP_X-Sendfile-Type X-Accel-Redirect;
        fastcgi_param HTTP_X-Accel-Mapping /udisk=/code/edusoho/app/data/udisk;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 8 128k;
    }

    # 配置设置图片格式文件
    location ~* \.(jpg|jpeg|gif|png|ico|swf)$ {
        # 过期时间为3年
        expires 3y;
        # 关闭日志记录
        access_log off;
        # 关闭gzip压缩，减少CPU消耗，因为图片的压缩率不高。
        gzip off;
    }
    # 配置css/js文件
    location ~* \.(css|js)$ {
        access_log off;
        expires 3y;
    }
    # 禁止用户上传目录下所有.php文件的访问，提高安全性
    location ~ ^/files/.*\.(php|php5)$ {
        deny all;
    }

    # 以下配置允许运行.php的程序，方便于其他第三方系统的集成。
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        fastcgi_param  HTTPS              off;
    }
}
```

2、下载edusoho,并授权文件夹

```shell
wget http://download.edusoho.com/edusoho-8.2.17.tar.gz
tar xf edusoho-8.2.17.tar.gz
chown -R www.www edusoho
```

3、调整php的上传大小(上传文件默认有限制大小)

```shell
[root@web01 ~]# vim /etc/php.ini
post_max_size = 200M
upload_max_filesize = 200M
[root@web01 code]# systemctl restart php-fpm
```

## 六、各开源项目网站

phpmyadmin	https://www.phpmyadmin.net/
zblog	https://www.zblogcn.com/
wordpress	https://cn.wordpress.org/
wecenter	http://www.wecenter.com/downloads/
edusohu	http://www.edusoho.com/open/show
