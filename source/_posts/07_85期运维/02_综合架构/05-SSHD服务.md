---
title: 05-Openssh服务
date: 2024-4-28 16:32:52
categories:
- 运维
- （二）综合架构
tags:
---

# 一、Openssh服务

## 1.1. openssh服务介绍

- 用于实现加密的远程连接/传输数据.
  openssh-server 服务端（sshd,/etc/ssh/sshd_config）
  openssh-clients客户端（命令 scp,ssh） 

## 1.2 telnet VS openssh

| 服务         | 共同点   | 区别       | 应用场景                                 |
| ------------ | -------- | ---------- | ---------------------------------------- |
| openssh 服务 | 远程连接 | 数据加密的 | 默认使用openssh                          |
| telnet 服务  | 远程连接 | 数据未加密 | 升级openssh服务的时候,启动telnet服务即可 |

telnet服务的使用

```shell
# 安装服务
yum install -y telnet-server

# 启动
systemctl start telnet.socket
# systemctl disable telnet.socket

#本地shell中连接
telnet 10.0.0.61 23
```



## 1.3 openssh的配置文件（*）

核心配置文件: `/etc/ssh/sshd_config`

| Openssh服务端配置详解   |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| 连接加速项目            |                                                              |
| UseDNS no               | 是否开启反向解析:ip ---> 域名或主机名                        |
| GSSAPIAuthentication no | GSS认证功能关闭                                              |
| 安全优化项目            |                                                              |
| Port                    | 默认是Port 22 端口范围1-65535 推荐1w以上的端口               |
| PermitRootLogin         | 禁用root用户 远程 登录权限. 默认是yes(可以让root远程登录) (ubt系统中默认是no) 使 用建议:先添加普通用户配置sudo权限,然后再禁用. |
| ListenAddress           | 监听的地址(后面需要指定本地网卡的ip地址) 可以控制用户只能通过内网访问,应用建议:一般配合着堡 垒机,vp |

  >操作建议:未来可以设置公网的端口与局域网端口不同(内外网端口分离)  
  >
  >```shell
  >ListenAddress 0.0.0.0:52113 #表示ssh都可以使用52113 无论局域网还是公网
  >ListenAddress 172.16.1.61:22 #表示ssh只有在172.16.1.61 局域网可以使用22端口
  >ssh root@172.16.1.61
  >ssh -p 52113 root@10.0.0.61
  >ssh -p 22 root@10.0.0.61
  >```



## 1.4 Openssh-Clients客户端命令  

### 1.4.1 scp  

使用方法如下

```shell
scp 文件/目录 用户名@ip:路径
-r 递归传输,传输目录
-p 保持属性信息不变
-P(大写) 指定端口号

# 案例
scp -rp -P 22 /etc/hostname root@10.0.0.41:/tmp/
```

### 1.4.2 ssh 

用于：

1. 远程连接.
2. 远程连接并执行命令或脚本.(不要执行交互式命令)  

案例01: 使用oldboy用户远程连接到10.0.0.41的22端口  

```shell
ssh -p 22 oldboy@10.0.0.41
```

案例02: 使用oldboy用户远程连接到10.0.0.41的22端口并执行whoami命令或ip a 命令  

```shell
ssh -p 22 oldboy@10.0.0.41 whoami
```

案例03: 远程连接10.0.0.31节点并执行多条命令:whoami , pwd, hostname命令  

```shell
ssh -p22 nfs01 "whoami && pwd && hostname -I "
# 或者用分号
ssh -p22 nfs01 "whoami ; pwd ; hostname -I "
```

>&&并且，命令行中表示前一个命令执行成功再执行后面的命令.
>; 分号，分隔命令.相当于是1行的结束  

### 1.4.3 sftp

ftp文件传输协议,服务和客户端,服务端端口是21和20.
openssh (sshd)也提供了,ftp功能,sftp,端口是22.
ftp客户端:常用sftp命令,软件xftp,winscp....  