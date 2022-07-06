---
title: 运维之综合架构--05--SSH服务器搭建
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---



## 一、SSH服务器介绍

>SSH是一个安全协议，在进行数据传输时，会对数据包进行加密处理，加密后在进行数据传输。确保了数据传输安全。
>

![ssh免密登录](04-ssh服务跳板机搭建.assets/ssh免密登录.png)

### 1 ssh的功能

1. 提供远程连接服务器的服务
2. 对传输的数据进行加密

### 2 常用服务的端口

- ftp -- tcp/20  tcp/21
- dns -- tcp/53  udp/53
- ssh  --  tcp/22
- telnet -- tcp/23 
- mysql -- tcp/3306
- http -- tcp/80
- https -- tcp/443

### 3 telnet服务搭建

安装并启动telnet服务

```shell
[root@backup ~]# yum install telnet-server -y
[root@backup ~]# systemctl start telnet.socket 
[root@backup ~]# useradd gs
[root@backup ~]# echo "1" | passwd --stdin gs
```

使用终端软件通过telnet使用gs用户登录连接

### 4 telnet与ssh的对比

| 服务连接方式 | 服务数据传输 | 服务监听端口 | 服务登陆用户     |
| ------------ | ------------ | ------------ | ---------------- |
| ssh          | 加密         | tcp/22       | 默认支持root登录 |
| telnet       | 明文         | tcp/23       | 不支持root登录   |

## 二、ssh与scp的使用

### 1 使用ssh 

```shell
ssh命令
	ssh 172.16.1.31					#取决当前执行此命令的用户
	ssh root@172.16.1.31			#标准的写法
	ssh -p22 root@172.16.1.31		#带端口
```

### 2 使用scp

```shell
scp 
# -P 指定端口，默认22端口可不写
# -r 表示递归拷贝目录
# -p 表示在拷贝文件前后保持文件或目录属性不变
# -l 限制传输使用带宽(默认kb) /8 ->KB  /1024  ->MB 
```

## 三、使用ssh密钥登录

```shell
1.生成密钥对    公钥  私钥
[root@m01 ~]# ssh-keygen -C 7242xxxxx@qq.com
	
2.将公钥推送到你需要连接的主机，第一次需要输入对端主机的密码
[root@m01 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.31
root@172.16.1.31's password:
	
3.通过ssh命令测试连接是否需要密码
[root@m01 ~]# ssh 172.16.1.31
Last login: Wed Jan  9 10:39:16 2019 from 172.16.1.61
```

## 四、ssh安全

```shell
[root@jumpserver ~]# vim /etc/ssh/sshd_config
[root@jumpserver ~]# systemctl restart sshd
```

SSH作为远程连接服务，通常我们需要考虑到该服务的安全，所以需要对该服务进行安全方面的配置。
1.更改远程连接登陆的端口		6666
2.禁止ROOT管理员直接登录		
3.密码认证方式改为密钥认证
4.重要服务不使用公网IP地址
5.使用防火墙限制来源IP地址
	
Port 6666                       # 变更SSH服务远程连接端口√
PermitRootLogin         no      # 禁止root用户直接远程登录√
PasswordAuthentication  no      # 禁止使用密码直接远程登录√
UseDNS                  no      # 禁止ssh进行dns反向解析，影响ssh连接效率参数√
GSSAPIAuthentication    no      # 禁止GSS认证，减少连接时产生的延迟√

## 五、防ssh暴力破解工具fail2ban

>fail2ban可以监控系统日志，并且根据一定规则匹配异常IP后使用Firewalld将其屏蔽，尤其是针对一些爆破/扫描等非常有效。

部署流程

```shell
1.开启Firewalld防火墙
[root@bgx ~]# systemctl start firewalld
[root@bgx ~]# systemctl enable firewalld
[root@bgx ~]# firewall-cmd --state
running

2.修改firewalld规则，启用Firewalld后会禁止一些服务的传输，但默认会放行常用的22端口, 如果想添加更多，以下是放行SSH端口（22）示例，供参考：
#放行SSHD服务端口
[root@bgx ~]# firewall-cmd --permanent --add-service=ssh --add-service=http 
#重载配
[root@bgx ~]# firewall-cmd --reload
#查看已放行端口
[root@bgx ~]# firewall-cmd  --list-service

3.安装fail2ban,需要有epel
[root@bgx ~]# yum install fail2ban fail2ban-firewalld mailx -y

4.配置fail2ban规则.local会覆盖.conf文件
[root@bgx fail2ban]# cat /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = 127.0.0.1/8
bantime  = 86400
findtime = 600
maxretry = 5
banaction = firewallcmd-ipset
action = %(action_mwl)s

[sshd]
enabled = true
filter  = sshd
port    = 22
action = %(action_mwl)s
logpath = /var/log/secure

5.启动服务，并检查状态
[root@bgx ~]# systemctl start fail2ban.service
[root@bgx ~]# fail2ban-client status sshd

6.清除被封掉的IP地址
[root@bgx ~]# fail2ban-client set sshd unbanip 10.0.0.1
```

