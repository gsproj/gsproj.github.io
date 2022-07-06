---
title: 运维之综合架构--07--Nginx(八)四层负载均衡
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、Nginx四层负载均衡介绍

四层负载均衡：（OSI传输层   ip:port）
	nginx1.9 版本加入
硬件：F5
软件：LVS、Haproxy、Nginx

1.四层+七层来作负载均衡，4层可以保证7层的负载均衡的高可用性。如:nginx就无法保证自己的服务高可用，需要依赖lvs或者keepalive来作。

2.如:tcp协议的负载均衡，有些请求是TCP协议的(mysql、ssh)，或者说这些请求只需要使用4层进行端口的转发就可以了，所以使用4层负载均衡。
	比如做：mysql读的负载均衡（轮询）
	比如做：端口映射、端口转发         tcp/udp

四层负载均衡总结
1.四层负载均衡仅能转发TCP/IP协议、UDP协议，通常用来转发端口，如: tcp/3306，tcp/22，udp/53。
2.四层负载均衡可以用来解决七层负载均衡的端口限制问题。（七层负载均衡最大使用65535个端口号）
3.可以用来解决七层负载均衡的高可用问题。（多台后端七层负载均衡能同时的使用）
4.四层的转发效率比七层的高的多，但仅支持tcp/ip协议，不支持http或者https协议

![image-20210823230415237](12-Nginx(八)Nginx四层负载均衡.assets/image-20210823230415237.png)

## 二、四层负载均衡配置

### 2.1 环境准备

| 服务器名                  | 公网IP   | 内网IP     |
| ------------------------- | -------- | ---------- |
| lb4-01                    | 10.0.0.3 |            |
| lb01                      | 10.0.0.5 | 172.16.1.5 |
| lb02                      | 10.0.0.6 | 172.16.1.6 |
| web、nfs、mysql服务器若干 |          |            |

### 2.2 新增lb02负载均衡服务器

1、准备`lb02`，安装Nginx服务

```shell
[root@lb02 ~]# cat /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
[root@lb02 ~]#  yum install nginx -y
```

2、拷贝`lb01`的 Nginx配置到`lb02`

```shell
[root@lb02 ~]# scp -rp root@172.16.1.5:/etc/nginx /etc/
```

3、启动Nginx

```shell
[root@lb02 conf.d]# nginx -t
[root@lb02 conf.d]# systemctl start nginx
[root@lb02 conf.d]# systemctl enable nginx
```

>报错：
>
>[root@lb02 ~]# nginx -t
>nginx: [emerg] unknown directive "check" in /etc/nginx/conf.d/proxy_node.conf:4
>nginx: configuration file /etc/nginx/nginx.conf test failed
>[root@lb02 ~]# vim /etc/nginx/conf.d/proxy_node.conf
>[root@lb02 ~]# nginx -t
>nginx: [emerg] unknown directive "check_status" in /etc/nginx/conf.d/proxy_node.conf:17
>nginx: configuration file /etc/nginx/nginx.conf test failed
>
>解决方法：
>
>因为lb01安装过第三方健康检查工具，而lb02上没有，先将这些报错点注释掉

4、测试lb02是否正常

将hosts中的10.0.0.5换成10.0.0.6，phpadmin、wordpress网页可以正常访问,
但是wecenter异常502 Bad Gateway
解决方法: 将web01、02上的的php.ini中的session.auto_start改为0,不然会502

```shell
[root@web01 php]# vim /etc/php.ini
session.auto_start = 0
[root@web01 php]# systemctl restart php-fpm.service
```

>网上搜索的解释：
>日常开发中，php.ini配置session.auto_start=0默认关闭会话时如果想开启会话需要调用session_start：
>session.auto_start 开启就自动完成了session_start()
>区别就在于在用SESSION前是否需要session_start();
>当session.auto_start = on时，执行 session_start() 将产生新的 session_id
>session.auto_start = on 的优点在于，任何时候都不会因忘记执行 session_start() 或 session_start() 在程序里的位置不对，而导致错误
>缺点在于，如果你使用的是第三方代码，则必须删去其中的全部 session_start() 。否则将不能得到正确的结果

### 2.3 配置四层负载均衡

1、新增`lb4-01`服务器，安装nginx

>nginx需要带--with-stream模块

```she
[root@lb4-01 ~]# cat /etc/yum.repos.d/nginx.repo 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
[root@lb4-01 ~]#  yum install nginx -y	
[root@lb4-01 ~]# vim /etc/nginx/nginx.conf
events {
    ....
}

include /etc/nginx/conf.c/*.conf;

http {
	.....
}
```

2、创建四层负载均衡配置的目录

```shell
[root@lb4-01 conf.c]# rm -f /etc/nginx/conf.d/default.conf   #删除http的80端口
[root@lb4-01 ~]# mkdir /etc/nginx/conf.c
[root@lb4-01 ~]# cd /etc/nginx/conf.c
[root@lb4-01 conf.c]# cat lb_domain.conf 
stream {
    upstream lb {
        server 172.16.1.5:80 weight=5 max_fails=3 fail_timeout=30s;
        server 172.16.1.6:80 weight=5 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 80;
        proxy_connect_timeout 3s;
        proxy_timeout 3s;
        proxy_pass lb;
    }
}
```

3、重载服务

```shell
[root@lb4-01 conf.c]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@lb4-01 conf.c]# systemctl restart nginx
[root@lb4-01 conf.c]# systemctl enable nginx
```

>报错：
>	[root@lb4-01 nginx]# nginx  -t
>	nginx: [emerg] unknown directive "stream" in /etc/nginx/conf.c/lb_domain.conf:1
>	nginx: configuration file /etc/nginx/nginx.conf test failed
>解决方法：
>	yum install nginx-mod-stream.x86_64 -y

>报错：
>[root@lb4-01 nginx]# systemctl status nginx
>nginx.service - The nginx HTTP and reverse proxy server
>Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
>Active: failed (Result: exit-code) since Tue 2021-08-24 07:48:42 CST; 13s ago
>Process: 2969 ExecStart=/usr/sbin/nginx (code=exited, status=1/FAILURE)
>Process: 2967 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
>Process: 2965 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
>Main PID: 1959 (code=exited, status=0/SUCCESS)
>
>Aug 24 07:48:40 lb4-01 nginx[2969]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
>Aug 24 07:48:40 lb4-01 nginx[2969]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
>Aug 24 07:48:41 lb4-01 nginx[2969]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
>Aug 24 07:48:41 lb4-01 nginx[2969]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
>Aug 24 07:48:42 lb4-01 nginx[2969]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
>Aug 24 07:48:42 lb4-01 nginx[2969]: nginx: [emerg] still could not bind()
>解决方法：
>	需要将[root@lb4-01 nginx]# vim /etc/nginx/nginx.conf
>	中默认的80端口改掉，比如改成81,这样才不冲突（奇怪。。）

4、访问测试

将Hosts中的对应网站地址的IP改成10.0.0.3，尝试访问，可以正常通过四层负载均衡来调度10.0.0.5和10.0.0.6七层负载均衡

![image-20210824001613717](12-Nginx(八)Nginx四层负载均衡.assets/image-20210824001613717.png)

![image-20210824001638506](12-Nginx(八)Nginx四层负载均衡.assets/image-20210824001638506.png)

## 三、四层负载均衡日志

### 3.1 配置过程

在四层负载均衡中配置Nginx

```shell
[root@lb4-01 ~]# cat /etc/nginx/conf.c/lb_domain.conf
stream {
    log_format  proxy '$remote_addr $remote_port - [$time_local] $status $protocol '
                  '"$upstream_addr" "$upstream_bytes_sent" "$upstream_connect_time"' ;
    access_log /var/log/nginx/proxy.log proxy;

    upstream lb {
....
```

### 3.2 查看日志

网页访问后，可见产生的日志

```shell
[root@lb4-01 ~]# tail -f /var/log/nginx/proxy.log
10.0.0.1 56396 - [25/Aug/2021:19:25:33 +0800] 200 TCP "172.16.1.5:80" "15150" "0.000"
10.0.0.1 59800 - [25/Aug/2021:19:25:34 +0800] 200 TCP "172.16.1.6:80" "8327" "0.000"
10.0.0.1 63468 - [25/Aug/2021:19:25:34 +0800] 200 TCP "172.16.1.5:80" "17882" "0.000"
10.0.0.1 50394 - [25/Aug/2021:19:25:35 +0800] 200 TCP "172.16.1.5:80" "24490" "0.000"
10.0.0.1 55549 - [25/Aug/2021:19:25:35 +0800] 200 TCP "172.16.1.5:80" "13201" "0.000"
10.0.0.1 64324 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.6:80" "70995" "0.000"
10.0.0.1 64566 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.6:80" "72650" "0.000"
10.0.0.1 65216 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.6:80" "68171" "0.000"
10.0.0.1 49668 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.5:80" "82071" "0.004"
10.0.0.1 49745 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.5:80" "63941" "0.001"
10.0.0.1 50574 - [25/Aug/2021:19:25:42 +0800] 200 TCP "172.16.1.6:80" "103581" "0.000"
```

## 三、使用nginx四层负载均衡实现tcp的转发（跳板）

实现跳板机的功能，比如：

	请求负载均衡 5555    --->     172.16.1.7:22;
	请求负载均衡 6666    --->     172.16.1.51:3306;

### 3.1 配置过程

配置Nginx

```shell
[root@lb4-01 ~]# cat /etc/nginx/conf.c/lb_domain.conf 
stream {
log_format  proxy '$remote_addr $remote_port - [$time_local] $status $protocol '
                  '"$upstream_addr" "$upstream_bytes_sent" "$upstream_connect_time"' ;
    access_log /var/log/nginx/proxy.log proxy;

#定义转发ssh的22端口
	upstream ssh_7 {
		server 10.0.0.7:22;
	}
#定义转发mysql的3306端口
	upstream mysql_51 {
		server 10.0.0.51:3306;
	}
    server {
        listen 5555;
        proxy_connect_timeout 3s;
        proxy_timeout 300s;
        proxy_pass ssh_7;
    }

    server {
        listen 6666;
        proxy_connect_timeout 3s;
        proxy_timeout 3s;
        proxy_pass mysql_51;
    }
}
```

测试访问

```shell
# 连接web01
ssh root@10.0.0.3 -p 5555

# 连接db01
ssh root@10.0.0.3 -p 6666
```









