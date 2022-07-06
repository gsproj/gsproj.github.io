---
title: 运维之综合架构--07--Nginx(一)安装与配置
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、Nginx简介

参考网站：https://zhuanlan.zhihu.com/p/266153320

## 二、Nginx安装

​	nginx有两种安装方式，yum安装和源码编译安装

### 2.1 yum安装（epel源）

```shell
vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1

yum clean all
yum makecache

yum install nginx -y
```

### 2.2 编译安装

查看yum安装的nginx的编译参数

```shell
[root@web01 ~]# nginx -V
nginx version: nginx/1.20.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1g FIPS  21 Apr 2020
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-compat --with-debug --with-file-aio --with-google_perftools_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_xslt_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```

源码获取

```shell
[root@web01 ~]# wget http://nginx.org/download/nginx-1.20.1.tar.gz
```

构建与编译

```shell
# 解压源码文件并进入文件夹内
tar -vxf nginx-1.20.1.tar.gz && cd nginx-1.20.1

# configure构建
./configure --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-compat --with-debug --with-file-aio --with-google_perftools_module --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_xslt_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-threads --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'

# 编译并安装
make -j 4 && make install
```

错误解决

```shell
# 错误1：
./configure: error: the invalid value in --with-ld-opt="-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E"
解决方法：
yum -y install redhat-rpm-config.noarch

# 错误2：
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
解决方法：
yum install pcre-devel -y

# 错误3：
./configure: error: SSL modules require the OpenSSL library.
You can either do not enable the modules, or install the OpenSSL library
into the system, or build the OpenSSL library statically from the source
with nginx by using --with-openssl=<path> option.
解决方法：
yum install openssl openssl-devel -y

# 错误4：
./configure: error: the HTTP XSLT module requires the libxml2/libxslt
libraries. You can either do not enable the module or install the libraries.
解决方法：
yum install libxml2 libxml2-devel libxslt  libxslt-devel -y

# 错误5：
./configure: error: the HTTP image filter module requires the GD library.
You can either do not enable the module or install the libraries.
解决方法：
yum -y install gd-devel

# 错误6：
./configure: error: perl module ExtUtils::Embed is required

解决方法： 
yum -y install perl-devel perl-ExtUtils-Embed

# 错误7：
./configure: error: the Google perftools module requires the Google perftools library
解决方法：
yum install gperftools-devel.x86_64 gperftools-libs.x86_64 gperftools.x86_64 -y
```

### 2.3 启动和停止服务

启动服务

```shell
# 先关闭httpd服务，防止冲突
[root@web01 ~]# systemctl stop httpd
[root@web01 ~]# systemctl disable httpd

# 再启动nginx服务
[root@web01 ~]# systemctl enable nginx
[root@web01 html]# systemctl start nginx
# 另一种启动方式
nginx
```

停止服务

```shell
systemctl stop nginx 
# 或者
nginx -s stop 
```

重启/重载服务

```shell
systemctl restart/reload nginx
nginx -s restart/reload 
```

>PS：重载reload和重启restart的差别
>
>reload将会等服务进程执行完再重启，而restart则是强制重启

## 三、Nginx目录结构说明

查看nginx的目录结构

```shell
[root@web01 html]# rpm -ql nginx
```

参考链接：

https://zhuanlan.zhihu.com/p/137262519

## 四、Nginx配置文件说明

>http server location扩展了解项
>http{}层下允许有多个Server{}层，一个Server{}层下又允许有多个Location
>http{} 标签主要用来解决用户的请求与响应。
>server{} 标签主要用来响应具体的某一个网站。
>location{} 标签主要用于匹配网站具体URL路径。

主配置文件说明

```shell
[root@web01 html]# cat /etc/nginx/nginx.conf
---核心模块---
user nginx;	#nginx进程运行的用户
worker_processes auto;	#nginx工作的进程数量
error_log /var/log/nginx/error.log;	#nginx的错误日志【警告及其警告以上的都记录】
pid /run/nginx.pid;	#nginx进程运行后的进程id
---核心模块---

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

---事件模块---
events {
    worker_connections 1024; # 一个work进程的最大连接数
    use epool;				 #使用epool网络模型
}
---事件模块---

---http核心层模块---
http {
	# 日志格式定义
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;	# 访问日志

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;	# 长连接超时时间
    types_hash_max_size 4096;
    #gzip on;			#是否开启压缩功能			

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;	# 包含哪个目录下面的*.conf文件，用于写server

    server {
        listen       80;	# 监听端口
        listen       [::]:80;
        server_name  _;		# 域名
        root         /usr/share/nginx/html;

		#charset koi8-r;			#字符集

		location / {	 			#位置
			root   /usr/share/nginx/html;	#代码的主文件位置
			index  index.html index.htm;	#服务端默认返回给用户的文件
		}
		location /test {	 			#位置
			root   /code/test/123/;	#代码的主文件位置
			index  index.html index.htm;	#服务端默认返回给用户的文件
		}

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
---http核心层模块---
```

## 五、案例：搭建web网站

准备配置文件ds

```she
[root@web01 code]# cat /etc/nginx/conf.d/game.conf
server {
        listen 80;
        server_name game.oldboy.com;

        location / {
                root /code;
                index index.html;
        }
}
```

按照配置文件创建文件夹并放入html项目文件

```shell
mkdir /code
cp html5.zip /code
cd /code
unzip html5.zip

[root@web01 code]# ls
ceshi  game  html5.zip  img  index.html  __MACOSX  readme.txt
```

重启nginx服务

```shell
systemctl restart nginx
systemctl reload nginx
```

通过物理机浏览器浏览

```shell
# 修改hosts文件
10.0.0.7 game.oldboy.com
# 网页访问
game.oldboy.com
```

## 六、Nginx虚拟主机

Nginx配置虚拟主机有如下三种方式：

- 单主机多IP
- 单主机多端口
- 单主机多域名

### 6.1 基于主机多IP方式(不常用)

```shell
[root@web01 conf.d]# cat ip.conf 
server {
	listen 10.0.0.7:80;
	server_name _;

	location / {
		root /code_ip_eth0;
		index index.html;
	}
}

server {
	listen 172.16.1.7:80;
	server_name _;

	location / {
		root /code_ip_eth1;
		index index.html;
	}
}

2.根据配置创建目录
[root@web01 conf.d]# mkdir /code_ip_eth0
[root@web01 conf.d]# echo "Eth0" > /code_ip_eth0/index.html

[root@web01 conf.d]# mkdir /code_ip_eth1
[root@web01 conf.d]# echo "Eth1" > /code_ip_eth1/index.html

3.重启nginx服务
[root@web01 conf.d]# systemctl restart nginx

4.使用curl命令测试
[root@web01 ~]# curl 172.16.1.7
Eth1
[root@web01 ~]# curl 10.0.0.7
Eth0
```

### 6.2 基于主机多端口方式（多用于内部测试）

```shell
1.配置多端口的虚拟主机
[root@web01 conf.d]# vim port.conf
server {
        listen 81;
        
        location / { 
                root /code_81;
                index index.html;
        }
}
server {
        listen 82;
        
        location / { 
                root /code_82;
                index index.html;
        }
}		

2.根据配置文件创建所需的目录
[root@web01 conf.d]# mkdir /code_8{1..2}
[root@web01 conf.d]# echo "81" > /code_81/index.html
[root@web01 conf.d]# echo "82" > /code_82/index.html

3.检查语法并重启服务
[root@web01 conf.d]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web01 conf.d]# systemctl restart nginx


4.如何去访问
	http://10.0.0.7:82/
```

### 6.3 基于主机多域名方式（常用）

```shell
1.准备多虚拟主机配置文件
[root@web01 conf.d]# cat test1.oldboy.com.conf 
server {
	listen 80;
	server_name test1.oldboy.com;

	location / {
		root /code/test1;
		index index.html;
	}
}

[root@web01 conf.d]# cat test2.oldboy.com.conf 
server {
	listen 80;
	server_name test2.oldboy.com;

	location / {
		root /code/test2;
		index index.html;
	}
}

2.根据配置文件创建对应的目录
[root@web01 conf.d]# mkdir /code/test{1..2} -p
[root@web01 conf.d]# echo "test1_server" > /code/test1/index.html
[root@web01 conf.d]# echo "test2_server" > /code/test2/index.html
[root@web01 conf.d]# nginx -t
[root@web01 conf.d]# systemctl restart nginx

3.配置域名解析
10.0.0.7      test1.oldboy.com
10.0.0.7      test2.oldboy.com

4.通过浏览器访问该网站
```

## 七、日志与错误排查

### 7.1 nginx配置文件自查

1.修改完配置记得检查语法

```
nginx -t
```

2.如果没有检查语法，直接重载导致报错，可查看错误信息

```shell
systemctl status nginx -l 
```

### 7.2 访问日志

可以为server和location单独设置访问日志（***涉及日志作用域***）

```shell
server {
    listen 80;
    server_name code.oldboy.com;
    
    # 将当前的server网站的访问日志记录至对应的目录，使用main格式
    access_log /var/log/nginx/code.oldboy.com.log main;
    location / {
        root /code;
    }
	
    # 当有人请求改favicon.ico时，不记录日志
    location /favicon.ico {
        access_log off;  # off 关闭
        return 200;
    }
}
```

访问日志参数详解

```shell
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

| **变量名称**            | **变量描述**                                 | **举例说明**                                                 |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| $remote_addr            | 客户端地址                                   | 113.140.15.90                                                |
| $remote_user            | 客户端用户名称                               | –                                                            |
| $time_local             | 访问时间和时区                               | 18/Jul/2012:17:00:01 +0800                                   |
| $request                | 请求的URI和HTTP协议                          | “GET /pa/img/home/logo-alipay-t.png HTTP/1.1″                |
| $http_host              | 请求地址，即浏览器中你输入的地址（IP或域名） | img.alipay.com10.253.70.103                                  |
| $status                 | HTTP请求状态                                 | 200                                                          |
| $upstream_status        | upstream状态                                 | 200                                                          |
| $body_bytes_sent        | 发送给客户端文件内容大小                     | 547                                                          |
| $http_referer           | 跳转来源                                     | “[https://cashier.alip](https://cashier.alip/)ay.com…/”      |
| $http_user_agent        | 用户终端代理                                 | “Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C; |
| $ssl_protocol           | SSL协议版本                                  | TLSv1                                                        |
| $ssl_cipher             | 交换数据中的算法                             | RC4-SHA                                                      |
| $upstream_addr          | 后台upstream的地址，即真正提供服务的主机地址 | 10.228.35.247:80                                             |
| $request_time           | 整个请求的总时间                             | 0.205                                                        |
| $upstream_response_time | 请求过程中，upstream响应时间                 | 0.002                                                        |

### 7.3 错误日志

```shell
tail -f /var/log/nginx/error.log
```

### 7.4 日志切割logrotate

配置文件，一般不需要修改，默认就行

```shell
[root@web01 logrotate.d]# cat /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    create 0640 nginx root	 	
    daily	# 每天切割日志
    rotate 10
    missingok	# 日志丢失忽略
    notifempty
    compress	# 日志文件压缩
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```





