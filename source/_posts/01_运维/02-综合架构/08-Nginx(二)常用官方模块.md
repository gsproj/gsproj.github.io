---
title: 运维之综合架构--07--Nginx(二)常用官方模块
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

>>>>本篇主要介绍Nginx的常用官方模块

## 一、目录索引-autoindex

### 1.1 使用方法1

按此方法设置后，访问网页http://module.test.com将显示文件目录

实际目录位于: /module

a.准备配置文件

```shell
[root@web01 module]# cat /etc/nginx/conf.d/autoindex.conf
server {
	listen 80;
	server_name module.test.com;
	charset utf-8,gbk; # 解决中文乱码

	location / {
		root /module;
		autoindex on;	# 开启目录索引
		autoindex_exact_size off;	# 显示文件大小，默认为on显示字节，off显示大概单位
		autoindex_localtime on;	# 默认off显示UTC时间，on显示本地时间
	}
}
```

b.准备对应的目录，并往目录中添加文件

```shell
mkdir /module/{centos,ubuntu,redhat}/ -p
```

c.检查语法并重新加载nginx

```shell
nginx -t 
systemctl restart nginx
```

### 1.2 使用方法2（推荐）

按此方法设置后，访问网页http://www.module.test.com/download将显示文件目录,主页可正常访问

实际目录位于：/module/download

```shell
[root@web01 module]# cat /etc/nginx/conf.d/autoindex.conf 
server {
	listen 80;
	server_name module.oldboy.com;
	charset utf-8,gbk;

	location / {	# 主页可以正常访问
		root /code;
		index index.html index.htm;
	}

	location /download {	# 当访问http://xxxx/download则访问/module/download文件夹，显示目录索引
		root /module; # 此时，访问的文件夹是/module/download
		autoindex on;
		autoindex_exact_size off;
		autoindex_localtime on;
	}
}
```

## 二、 状态监控页面-stub_status

>需要nginx附带--with-http_stub_status_module模块才能使用

### 2.1 使用方法

a.设置配置文件

```shell
在/etc/nginx/conf.d/autoindex.conf里面附加内容
location /nginx_status {
	stub_status;
}
```

b.重启Nginx服务

```shell
重启nginx服务
systemctl reload nginx
```

c.网页访问测试

```shell
访问:
http://module.test.com/nginx_status
网页显示：
Active connections: 2 
server accepts handled requests
		3 			3 	33 
Reading: 0 Writing: 1 Waiting: 1 
```

d.通过选项关闭长连接

```shell
# 注意, 一次TCP的连接，可以发起多次http的请求, 如下参数可配置进行验证
keepalive_timeout  0;   # 类似于关闭长连接
keepalive_timeout  65;  # 65s没有活动则断开连接
```

```shell
在/etc/nginx/conf.d/autoindex.conf里面的附加内容修改为
location /nginx_status {
	stub_status;
	keepalive_timeout  0; 
}
```

e.再次测试网页访问

```shell
关闭长连接后的显示：
Active connections: 2 
server accepts handled requests
 21 21 20 
Reading: 0 Writing: 1 Waiting: 1 
# 可见每次HTTP请求都要重新发起TCP连接
```

### 2.2 监控页面内容解释

| 参数项             | 作用                                        |
| ------------------ | ------------------------------------------- |
| Active connections | 当前活动客户端连接数，包括Waiting等待连接数 |
| accepts            | 已接受总的TCP连接数                         |
| handled            | 已处理总的TCP连接数                         |
| requests           | 客户端总的http请求数                        |
| Reading            | 当前nginx读取请求头的连接数                 |
| Writing            | 当前nginx将响应写回客户端的连接数           |
| Waiting            | 当前等待请求的空闲客户端连接数              |

## 三、基于IP的访问控制

>某网页内的数据比较重要，怎么控制那些人可以访问，那些人不能访问呢？

可以来源的IP地址做限制，常用的三种控制方法：

- 拒绝10.0.0.1来源IP访问，其他人允许

  ```shell
  location /nginx_status {
      stub_status;
      deny 10.0.0.1/32;
      allow all;
  }
  ```

- 允许10.0.0.1来源IP访问，其他人全部拒绝

  ```shell
  location /nginx_status {
      stub_status;
      allow 10.0.0.1/32;
      deny all;
  }
  ```

- 实际配置监控Nginx状态时，仅允许该服务器的回环地址访问127.0.0.1

  ```shell
  # 最安全
  location /nginx_status {
      stub_status;
      allow 127.0.0.1;
      deny all;
  }
  ```

## 四、基于密码的身份验证

>重要数据网站，要实现需要用户名密码认证，怎么做呢？

1. 生成一个密码文件，密码文件的格式  name:password(加密)  （建议使用htpasswd）  openssl password

   ```shell
   [root@web01 conf.d]# yum install httpd-tools -y
   [root@web01 conf.d]# htpasswd -c -b /etc/nginx/auth_conf oldboy oldboy
   [root@web01 conf.d]# cat /etc/nginx/auth_conf
   oldboy:$apr1$Kp87VSae$658Nt5bm4iiblQkUvP7u61
   ```

2. 配置Nginx，限制对应的资源

   ```shell
   	location /download {
   		root /module;
   		autoindex on;
   		autoindex_exact_size off;
   		autoindex_localtime on;
   		
   		auth_basic "Please Password!!!";
   		auth_basic_user_file /etc/nginx/auth_conf; 
   		# 注意认证文件路径，否则网页认证后会403，可是为什么放其他目录就不行？
   	}
   ```

## 五、Nginx连接限制

>网站请求数太多，不堪重负了，怎么保障部分用户能够正常访问

### 5.1 限制连接数

`设置共享内存区域和给定键值的最大允许连接数。超过此限制时，服务器将返回错误以回复请求`

- 编辑配置文件

  ```shell
  # http标签段定义连接限制
  http{
      limit_conn_zone $binary_remote_addr zone=conn_zone:10m;
  }
  server {
      # 同一时刻只允许一个客户端连接
      limit_conn conn_zone 1; 
  
      location / {
          root /code;
          index index.html;
      }
  ```

- 使用ab工具进行压力测试

  ````shell
  # 使用ab工具进行压力测试
  [root@xuliangwei ~]# yum install -y httpd-tools
  [root@xuliangwei ~]# ab -n 500 -c 2  http://127.0.0.1/index.html
  # 可见500次中有失败的请求
  ````

- 查看拦截日志

  ```shell
  tail -f /var/log/nginx/error.log
  类似于
  2019/01/14 11:11:22 [error] 29962#29962: *19 limiting connections by zone "conn_zone", client: 47.110.176.164, server: www.xuliangwei.com, request: "GET / HTTP/1.1", host: "www.xuliangwei.com"
  2019/01/14 11:11:23 [error] 29962#29962: *19 limiting connections by zone "conn_zone", client: 47.110.176.164, server: www.xuliangwei.com, request: "GET / HTTP/1.1", host: "www.xuliangwei.com"
  2019/01/14 11:11:25 [error] 29962#29962: *19 limiting connections by zone "conn_zone", client: 47.110.176.164, server: www.xuliangwei.com, request: "GET / HTTP/1.1", host: "www.xuliangwei.com"
  2019/01/14 11:11:25 [error] 29962#29962: *19 limiting connections by zone "conn_zone", client: 47.110.176.164, server: www.xuliangwei.com, request: "GET / HTTP/1.1", host: "www.xuliangwei.com"
  ```


### 5.2 限制请求数（更精准）

`设置共享内存区域和请求的最大突发大小。过多的请求被延迟，直到它们的数量超过最大突发大小，在这种情况下请求以错误终止。默认情况下，最大突发大小等于零。`

- 定义限制的Key

  ```shell
  [root@web01 conf.d]# cat test1.oldboy.com.conf 
  limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;
  server {
  	listen 80;
  	server_name test1.oldboy.com;
  	
  	limit_req zone=req_zone burst=5 nodelay;
  	limit_req_status 412;
  	error_page 412 /err.html;    #这个文件必须存在/code/test1/err.html
  
  	location / {
  		root /code/test1;
  		index index.html;
  	}
  }
  ```

- 填写hosts域名解析 （测试：域名访问才有效果，直接访问没效果）

  ```shell
  echo "10.0.0.7 test1.oldboy.com" >> /etc/hosts
  ```

- 压力测试

  ```shell
  ab -n 50 -c 20 http://test1.oldboy.com/index.html
  ```

- 查看错误日志

  ```shell
  2019/01/14 11:28:22 [error] 2073#2073: *3 limiting requests, excess: 5.737 by zone "req_zone", client: 10.0.0.1, server: test1.oldboy.com, request: "GET / HTTP/1.1", host: "test1.oldboy.com"
  2019/01/14 11:28:22 [error] 2073#2073: *3 limiting requests, excess: 5.611 by zone "req_zone", client: 10.0.0.1, server: test1.oldboy.com, request: "GET / HTTP/1.1", host: "test1.oldboy.com"
  2019/01/14 11:28:22 [error] 2073#2073: *3 limiting requests, excess: 5.450 by zone "req_zone", client: 10.0.0.1, server: test1.oldboy.com, request: "GET / HTTP/1.1", host: "test1.oldboy.com"
  ```

## 六、Nginx匹配符和优先级

Location语法示例

```shell
location [=|^~|~|~*|!~|!~*|/] /uri/ { ...
}
```

匹配优先级

```shell
匹配符	匹配规则						优先级
=		精确匹配						1 	用到
^~		以某个字符串开头				2
~		区分大小写的正则匹配			3	常用
~*		不区分大小写的正则匹配			4
!~		区分大小写不匹配的正则			5
!~*		不区分大小写不匹配的正则		6
/		通用匹配，任何请求都会匹配到	7	常用
```

### 6.1 匹配案例

> 参考网站：https://blog.csdn.net/qq_41980405/article/details/111402208

- 通用匹配，任何请求都会匹配到

  ```nginx
  location / {
      ...
  }
  ```

- 严格区分大小写，匹配以.php结尾的都走这个location    

  ```nginx
  location ~ \.php$ {
      ...
  }
  ```

- 严格区分大小写，匹配以.jsp结尾的都走这个location 

  ```nginx
  location ~ \.jsp$ {
      ...
  }
  ```

- 不区分大小写匹配，只要用户访问.jpg,gif,png,js,css 都走这条location

  ```nginx
  location ~* .*\.(jpg|gif|png|js|css|mp4)$ {
      ...
  }
  ```

- 不区分大小写匹配

  ```nginx
  location ~* "\.(sql|bak|tgz|tar.gz|.git)$" {
      ...
  }
  ```

  

