---
title: 运维之综合架构--07--Nginx(七)均衡调度
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、Nginx均衡调度算法

Nginx七层负载均衡分为5种调度算法

| 调度算法          | 概述                                                         |
| ----------------- | ------------------------------------------------------------ |
| 轮询（常用）      | 按时间顺序逐一分配到不同的后端服务器(默认)                   |
| weight（面试点）  | 加权轮询,weight值越大,分配到的访问几率越高                   |
| ip_hash（面试点） | 每个请求按访问IP的hash结果分配,这样来自同一IP的固定访问一个后端服务器 |
| url_hash          | 按照访问URL的hash结果来分配请求,是每个URL定向到同一个后端服务器 |
| least_hash        | 最少链接数,那个机器链接数少就分发                            |

### 1.1 加权轮询

比如实现访问5次web01，1次web02

当web服务器配置不相同，有差距时，可以用此方法

在`lb01`配置

```shell
[root@lb01 nginx]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        server 172.16.1.7:80 weight=5;
        server 172.16.1.8:80 weight=1;
}

server {
...
```

### 1.2 ip_hash

>PS：不能与weight一起使用

根据请求的IP地址，固定访问到某一后端，除非已选择的后端down了，有点浪费资源

在`lb01`配置

```shell
[root@lb01 nginx]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        ip_hash;
        server 172.16.1.7:80;
        server 172.16.1.8:80;
}

server { 
...
```

## 二、Nginx负载均衡后端状态

| 状态                     | 概述                                                       |
| ------------------------ | ---------------------------------------------------------- |
| down                     | 当前的server暂时不参与负载均衡                             |
| backup                   | 预留的备份服务器                                           |
| max_conns                | 限制最大的接收连接数                                       |
| max_fails（健康检查）    | 允许请求失败的次数（不够精准，作用不大，得知道，面试会问） |
| fail_timeout（健康检查） | 经过max_fails失败后, 服务暂停时间                          |

### 2.1 down状态

>一般用于停机维护

在`lb01`配置，可见当两台web的Nginx服务都正常时，只能访问web02，当web02的Nginx服务挂了，返回502

```shell
[root@lb01 nginx]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        server 172.16.1.7:80 down;
        server 172.16.1.8:80;
}

server {
...
```

### 2.2 backup状态 

在`lb01`配置，可见当两台web都正常时，只能访问web02，如果web02的服务挂了，会访问web01，当web02恢复后，再次访问到的是web02

```shell
[root@lb01 nginx]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        server 172.16.1.7:80 backup;
        server 172.16.1.8:80;
}

server {
...
```

### 2.3 健康检查

>自带的健康检查不够精准，且看不到信息，面试会问

```shell
[root@lb01 nginx]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        server 172.16.1.7:80 max_fails=2 fail_timeout=10s;
        server 172.16.1.8:80 max_fails=2 fail_timeout=10s;
}

server {
...
```

## 三、第三方健康检查模块check_upstream

>检测更精准，且有页面可以展示服务端的状态，需要编译安装

### 3.1 编译安装Nginx

1、安装依赖包

```shell
[root@lb02 ~]# yum install -y gcc glibc gcc-c++ pcre-devel openssl-devel  patch libxml2 -libxml2-devel libxslt libxslt-devel gd-devel perl-ExtUtils-Embed gperftools-devel.x86_64 gperftools-libs.x86_64 gperftools.x86_64
```

2、下载Nginx源码及第三方模块源码

>PS：为保持一致，先通过yum源安装nginx，这是当前实验环境的Nginx版本
>
>[root@lb01 nginx]# nginx -version
>nginx version: nginx/1.20.1

```shell
[root@lb02 ~]# wget http://nginx.org/download/nginx-1.20.1.tar.gz
[root@lb02 ~]# wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/master.zip
```

3、解压nginx源码包以及第三方模块

```shell
[root@lb01 ~]# unzip master.zip
[root@lb01 ~]# tar -vxf nginx-1.20.1.tar.gz
```

4、打补丁

>打补丁(nginx的版本是1.20.1补丁就选择1.20.1的,p1代表在nginx目录，p0是不在nginx目录)

```shell
[root@lb01 nginx-1.20.1]# cd nginx-1.20.1/
[root@lb01 nginx-1.20.1]# patch -p1 < ../nginx_upstream_check_module-master/check_1.20.1+.patch
```

5、编译Nginx，附带模块参数

>通过nginx -V获取configure参数，尽量保持一致

--add-module=/root/nginx_upstream_check_module-master

```shell
[root@lb01 nginx-1.20.1]# ./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-compat --with-debug --with-google_perftools_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_xslt_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --add-module=/root/nginx_upstream_check_module-master
```

6、在已有的负载均衡上增加健康检查的功能

```shell
[root@lb01 nginx-1.20.1]# cat /etc/nginx/conf.d/proxy_node.conf
upstream node {
        server 172.16.1.7:80;
        server 172.16.1.8:80;
        check interval=3000 rise=2 fall=3 timeout=1000 type=tcp;
}
```

### 3.2 功能测试

1、正常情况下的检测数据

![image-20210821113447825](11-Nginx(七)Nginx负载均衡调度与健康检查.assets/image-20210821113447825.png)

2、测试将web01的nginx服务关掉

![image-20210821114011230](11-Nginx(七)Nginx负载均衡调度与健康检查.assets/image-20210821114011230.png)

3、日志分析

```shell
[root@lb01 nginx-1.20.1]# tail -f /var/log/nginx/error.log
# 两web服务正常，能获取peer
2021/08/21 02:42:11 [error] 50522#50522: enable check peer: 172.16.1.8:80
2021/08/21 02:42:12 [error] 50522#50522: enable check peer: 172.16.1.7:80
2021/08/21 02:46:17 [error] 50522#50522: *13 connect() failed (111: Connection refused) while connecting to upstream, client: 10.0.0.1, server: node.oldboy.com, request: "GET / HTTP/1.1", upstream: "http://172.16.1.7:80/", host: "node.oldboy.com"
# 停止web01的nginx服务
2021/08/21 02:46:20 [error] 50522#50522: disable check peer: 172.16.1.7:80
# 重新启动web01的nginx服务
2021/08/21 02:47:59 [error] 50522#50522: enable check peer: 172.16.1.7:80
```

## 四、如何解决网站重复登录的问题

有三种方法解决：

1. ip_hash -- 会造成某一台主机的压力过大
2. session复制
3. session共享
   1. 本地文件 --> nfs共享
   2. 通过程序，写入redis数据库（常用）
   3. 通过程序，写入mysql数据库

本案例，选用3.2配置，session共享，写入redis数据库

### 4.1 安装phpmyadmin重现问题

`web01`和`web02`都需要安装

1、配置Nginx

```shell
[root@web01 conf.d]# cat php.conf
server {
	listen 80;
	server_name php.oldboy.com;
	root /code/phpMyAdmin-4.8.4-all-languages;

	location / {
		index index.php index.html;
	}

	location ~ \.php$ {
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
[root@web01 conf.d]# systemctl restart nginx
```

2、安装phpmyadmin

```shell
[root@web01 conf.d]# cd /code
[root@web01 code]# wget https://files.phpmyadmin.net/phpMyAdmin/4.8.4/phpMyAdmin-4.8.4-all-languages.zip
[root@web01 code]# unzip phpMyAdmin-4.8.4-all-languages.zip
```

3、配置phpmyadmin连接远程数据库

```shell
[root@web01 code]# cd phpMyAdmin-4.8.4-all-languages/
[root@web01 phpMyAdmin-4.8.4-all-languages]# cp config.sample.inc.php config.inc.php
[root@web01 phpMyAdmin-4.8.4-all-languages]# vim config.inc.php
/* Server parameters */
$cfg['Servers'][$i]['host'] = '172.16.1.51';
```

4、配置授权

>这个文件夹中会记录session，需要权限
>
>session_start(): Failed to read session data: files (path: /var/lib/php/session)

```shell
[root@web01 conf.d]# chown -R www.www /var/lib/php/
```

5、将web01上配置好的phpmyadmin以及nginx的配置文件推送到web02主机上

```shell
[root@web01 code]# scp -rp  phpMyAdmin-4.8.4-all-languages root@172.16.1.8:/code/
[root@web01 code]# scp /etc/nginx/conf.d/php.conf  root@172.16.1.8:/etc/nginx/conf.d/
```

6、重载Nginx服务，授权访问权限

```shell
[root@web02 code]# systemctl restart nginx
[root@web02 code]# chown -R www.www /var/lib/php/
```

7、接入负载均衡，并重启nginx服务，`lb01`操作

```shell
[root@lb01 conf.d]# vim proxy_php.com.conf 
upstream php {
        server 172.16.1.7:80;
        server 172.16.1.8:80;
}
server {
        listen 80;
        server_name php.oldboy.com;
        location / {
                proxy_pass http://php;
                include proxy_params;
        }
}

[root@lb01 conf.d]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@lb01 conf.d]# systemctl restart nginx
```

9、测试访问

```shell
网页访问：
	http://php.oldboy.com
登录账户:
	采用db01（172.16.1.51）的mysql远程账户oldboy/Bgx123.com
此时访问phpmyadmin登录将一直失败，因为轮询访问，session一直变：
报错：
Failed to set session cookie. Maybe you are using HTTP instead of HTTPS to access phpMyAdmin.
```

### 4.2 解决问题

1、在`db01`安装redis数据库

```shell
[root@db01 ~]# yum install redis -y
```

2、配置并启动redis

```shell
[root@db01 ~]# sed  -i '/^bind/c bind 127.0.0.1 172.16.1.51' /etc/redis.conf
[root@db01 ~]# systemctl start redis
[root@db01 ~]# systemctl enable redis
```

3、`web01`的php配置session连接redis

```shell
#1.修改/etc/php.ini文件
[root@web01 ~]# vim /etc/php.ini
session.save_handler = redis
session.save_path = "tcp://172.16.1.51:6379"
;session.save_path = "tcp://172.16.1.51:6379?auth=123" #如果redis存在密码，则使用该方式
session.auto_start = 1
```

4、注释php-fpm.d/www.conf里面的两条内容，否则session内容会一直写入/var/lib/php/session目录中

```she
;php_value[session.save_handler] = files
;php_value[session.save_path]    = /var/lib/php/session
```

5、重启php-fpm

```shell
[root@web02 code]# systemctl restart php-fpm
```

6、将`web01`的配置文件推送到`web02`上

```she
[root@web01 code]# scp /etc/php.ini root@172.16.1.8:/etc/php.ini  
[root@web01 code]# scp /etc/php-fpm.d/www.conf root@172.16.1.8:/etc/php-fpm.d/www.conf 
```

7、重启服务

```she
[root@web02 code]# systemctl restart php-fpm
```

8、再次测试访问网站

可以登录，并且session的值将记录到redis数据库

```shell
[root@db01 ~]# redis-cli -h 172.16.1.51
172.16.1.51:6379> KEYS *
1) "PHPREDIS_SESSION:716e61cabeb40f974fcbcdcac65f8607"
2) "PHPREDIS_SESSION:42ca5a8ea9375a93a7974427b77b3dd7"
172.16.1.51:6379>
```

9、刷新页面的负载均衡效果展示

![image-20210823225246243](11-Nginx(七)Nginx负载均衡调度与健康检查.assets/image-20210823225246243.png)

![image-20210823225305632](11-Nginx(七)Nginx负载均衡调度与健康检查.assets/image-20210823225305632.png)

10、cookie保存到redis展示

![image-20210823225412626](11-Nginx(七)Nginx负载均衡调度与健康检查.assets/image-20210823225412626.png)
