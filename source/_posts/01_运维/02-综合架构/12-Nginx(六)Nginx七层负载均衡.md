---
title: 运维之综合架构--07--Nginx(六)七层负载均衡
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---

## 一、七层负载均衡简介(需补充)

### 1.1 nginx代理的局限性

​	一个location仅能代理后端一台主机

### 1.2 七层负载均衡

Nginx负载均衡
	负载
	负载均衡
	调度
	load balance
	LB
公有云
	SLB		阿里云负载均衡
	QLB		青云负载均衡
	CLB		腾讯负载均衡
	ULB		ucloud的负载均衡

### 1.3 四层负载均衡和七层负载均衡的区别

四层负载均衡数据包在底层就进行了分发，而七层负载均衡数据包则是在最顶层进行分发、由此可以看出，七层负载均衡效率没有四负载均衡高。
但七层负载均衡更贴近于服务，如:http协议就是七层协议，我们可以用Nginx可以作会话保持，URL路径规则匹配、head头改写等等，这些是四层负载均衡无法实现的。

## 二、配置实例

### 2.1 测试环境准备

| 主机名称 | 应用环境                    | 外网地址  | 内网地址    |
| -------- | --------------------------- | --------- | ----------- |
| web01    | nginx + php（提供网页服务） | 10.0.0.7  | 172.16.1.7  |
| web01    | nginx + php（提供网页服务） | 10.0.0.8  | 172.16.1.8  |
| lb01     | nginx（提供代理服务）       | 10.0.0.5  | 172.16.1.5  |
| db01     | mysql                       | 10.0.0.51 | 172.16.1.51 |

### 2.2 配置步骤

1、web01/02的网页服务搭建

```shell
[root@web01 ~]# cd /etc/nginx/conf.d/
[root@web01 conf.d]# cat node.conf 
server {
    listen 80;
    server_name node.oldboy.com;
    location / {
        root /node;
        index index.html;
    }
}
[root@web01 conf.d]# mkdir /node
[root@web01 conf.d]# echo "Web01..." > /node/index.html
[root@web01 conf.d]# systemctl restart nginx
```

2、配置nginx负载均衡

```shell
[root@lb01 ~]# cd /etc/nginx/conf.d/
[root@lb01 conf.d]# cat node_proxy.conf 
upstream node {
    server 172.16.1.7:80;
    server 172.16.1.8:80;
}
server {
    listen 80;
    server_name node.oldboy.com;

    location / {
        proxy_pass http://node;
        include proxy_params;
    }
}
[root@lb01 conf.d]# systemctl restart nginx
```

3、客户端网页测试访问，F5刷新，可见web01/web02在循环

![image-20210820132420586](/img/image-20210820132420586.png)

![image-20210820132431971](/img/image-20210820132431971.png)

### 2.3 负载均衡也可以使用代理参数

>与之前部署的知乎wencenter、博客mywordpress结合

1、准备Nginx负载均衡调度使用的proxy_params

```shell
[root@Nginx ~]# vim /etc/nginx/proxy_params
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;
```

2、负载均衡配置：

```shell
[root@lb01 conf.d]# vim proxy_oldboy.com.conf
upstream node {
        server 172.16.1.7:80;
        server 172.16.1.8:80;
}
server {
        listen 80;
        server_name blog.oldboy.com;
        location / {
                proxy_pass http://node;
                include proxy_params;
        }
}

server {
        listen 80;
        server_name zh.oldboy.com;
        location / {
                proxy_pass http://node;
                include proxy_params;
        }
}
```

### 2.4 负载均衡测试

正常情况下：

```shell
1、两台web服务器的nginx服务均打开（模拟流量分摊）
	可以负载均衡，轮流访问
2、关掉一台web服务器的nginx服务（模拟容灾）
	网页仍可以正常打开
3、两台web服务器的Nginx服务均关闭
	网页无法打开，502
```

存在一种情况，两个web的nginx服务都没挂，但是其中一台的php-fpm服务挂了，模拟这种场景

```shell
[root@web02 ~]# systemctl stop php-fpm
```

此时F5刷新wordpress的页面，将一会502，一会正常，体验不好，可通过添加nginx代理参数解决

```shell
[root@lb01 conf.d]# cat /etc/nginx/proxy_params
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_next_upstream error timeout http_500 http_502 http_503 http_504; # 解决问题

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffering on;
proxy_buffer_size 32k;
proxy_buffers 4 128k;
```

>PS：问题原因
>
>使用nginx负载均衡时，如何将后端请求超时的服务器流量平滑的切换到另一台上。
>Nginx是本身是有机制的，如果出现一个节点down掉的时候，Nginx会更据你具体负载均衡的设置，将请求转移到其他的节点上，但是，如果后台服务连接没有down掉，并且返回错误异常码了如：504、502、500，Nginx就会直接返回从后端获取的异常代码。
