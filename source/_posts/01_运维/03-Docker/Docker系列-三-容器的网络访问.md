---
title: Docker系列(三)-容器的网络访问
date: 2021-06-29 09:34:01
categories:
- 运维
- （三）Docker
tags:
---

## 一、容器的网络访问流程

>参考：https://z.itpub.net/article/detail/FE8EBAC62D5881E3A432291F8C8E4F02

### 1 虚拟机注意事项 

查看net.ipv4.ip_forward值是否为1

```shell
sysctl -a | grep ipv4 | grep forward 
```

只有值为1时docker容器才能上网，而vmware虚拟机挂起将使他变成0，解决方法：

- sysctl net.ipv4.ip_forward = 1 设置为1
- 不要挂起虚拟机，直接关机重启，docker服务在启动时会将它改为1

## 二、容器端口映射

### 1 docker run -p端口映射参数

指定端口访问

```shell
# 将容器80端口，映射到主机180端口
docker run -p 180:80
```

指定IP+端口访问

```shell
# 多网卡环境下，将容器80端口映射到10.0.0.1的180端口
docker run -p 10.0.0.1:180:80
```

指定随机端口

```she
# 将容器80端口，映射到主机随机端口
docker run -p 10.0.0.1::80
```

指定随机端口+UDP (默认映射TCP)

```shell
# 将容器80端口映射到主机随机端口，并使用UDP协议
-p 10.0.0.100:80:udp # 指定随机端口 + udp
```

可以指定多个端口

```shell
-p 180:80 -p 1443:443 # 指定多个端口
```

随机映射

```shell
docker run -P
```

