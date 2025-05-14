---
title: 01-Docker基础
date: 2025-05-06 13:52:52
categories:
- 学神K8S
- （一）Docker容器入门
tags:
---

# 一、虚拟机准备

## 1.1 虚拟机配置

模板机，2G内存、2核CPU、80G存储空间

网络两个网卡：

​	ens33: NAT模式，192.168.40.0/24网段，网关192.168.40.2

​	ens36: LAN区段，172.16.1.0/24网段

系统：RockyLinux 8.10

## 1.2 模板机初始化

1、基础初始化

```shell
# 1、关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 2、关闭selinux
sed -i 's#SELINUX=enforcing#SELINUX=disable#g' /etc/selinux/config
setenforce 0

# 3、安装常用软件
yum install -y vim tree wget bash-completion bashcompletion-extras lrzsz net-tools sysstat iotop iftop htop unzip nc nmap telnet bc psmisc httpd-tools bindutils

# 4、设置主机名
hostnamectl set-hostname muban
```

2、配置时间同步

编辑`vim /etc/chrony.conf`文件，添加时间同步服务器

```shell
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 210.72.145.44 iburst
server ntp.aliyun.com iburst
...
```

重启服务生效

```shell
systemctl restart chronyd.service
```

查看当前的时间同步服务器

```shell
[root@localhost yum.repos.d]# chronyc sources -y
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 210.72.145.44                 0   8     0     -     +0ns[   +0ns] +/-    0ns
^* 203.107.6.88                  2   6   377    30    -97us[ -564us] +/-   37ms
```

3、命令行颜色

```shell
# 写入到/etc/profile中
export PS1='[\[\e[34;1m\]\u@\[\e[0m\]\[\e[32;1m\]\H\[\e[0m\]\[\e[31;1m\] \w\[\e[0m\]]\$'
# 生效
source /etc/profile
```

4、一键修改IP和hostname

```shell
[root@muban /etc]#cat /usr/local/mytools/bin/setip.sh 
#!/bin/bash
#author: haris
#desc: change ip and hostname
#version: v1.0
#判断参数格式是否为2
[ $# -ne 2 ] && {
echo "脚本使用姿势不对"
echo "正确姿势:$0 主机名 ip地址"
echo "如:$0 muban 211"
        exit 1
}
#获取当前主机ip地址
ip=`hostname -I |awk '{print $1}'|sed 's#.*\.##g'`
#新的ip
ip_new=`echo $2 |sed 's#^.*\.##g'`
#新的主机名
hostname=$1
#修改ip
sed -i "s#192.168.40.$ip#192.168.40.$ip_new#g" /etc/sysconfig/network-scripts/ifcfg-ens33
sed -i "s#172.16.1.$ip#172.16.1.$ip_new#g" /etc/sysconfig/network-scripts/ifcfg-ens36
#重启网卡
systemctl restart NetworkManager
#修改主机名
hostnamectl set-hostname $hostname
```





