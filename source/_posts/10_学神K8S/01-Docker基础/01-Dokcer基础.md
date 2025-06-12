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

## 1.2 基础初始化

1、关闭防火墙安装基础软件

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

加到crontab里面定时执行

```shell
# 一分钟自动重启一次服务
* * * * * /usr/bin/systemctl restart chronyd
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

## 1.3 开启包转发功能

1、br_netfilter模块用于将桥接流量转发至iptables链，需要开启转发。

```shell
# 开启模块
modprobe br_netfilter

# 验证是否已加载
[root@master1 /etc/sysctl.d]#lsmod | grep br_netfilter
br_netfilter           22256  0 
bridge                151336  1 br_netfilter
```

2、创建配置文件，修改内核参数

```shell
# 创建配置文件
cat > /etc/sysctl.d/docker.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 执行命令，使内核参数生效
sysctl -p /etc/sysctl.d/docker.conf
```

3、自动加载模块

```shell
# 编辑配置文件
[root@master1 ~]#cat /etc/rc.sysint 
#!/bin/bash
for file in /etc/sysconfig/modules/*.modules; 
do
[ -x $file ] && $file
done

# 添加模块自动加载的文件
[root@master1 ~]#cat /etc/sysconfig/modules/br_netfilter.modules
modprobe br_netfilter

# 添加可执行权限
chmod 755 /etc/sysconfig/modules/br_netfilter.modules
```

重启docker生效

## 1.4 安装Docker

>在线联网安装，Docker-ce

1、配置docker-ce国内的yum源（阿里云）

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

2、安装docker依赖包

```shell
yum install -y device-mapper-persistent-data lvm2
```

3、安装docker-ce

```shell
yum install docker-ce -y
```

4、启动docker服务器

```shell
systemctl start docker && systemctl enable docker
```

5、查看docker版本

```shell
docker verison
```



## 2.5 配置镜像加速

在OS中通过配置文件配置镜像加速

>镜像加速器多写几个，不然容易超时拉取失败
>
>Error response from daemon: Get "https://registry-1.docker.io/v2/

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.imgdb.de",
    "https://docker-0.unsee.tech",
    "https://docker.hlmirror.com",
    "https://docker.1ms.run",
    "https://func.ink",
    "https://lispy.org",
    "https://docker.xiaogenban1993.com"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```







# 二、Docker基本概念

## 1.1 Docker常用概念

**镜像（image）**：一个可运行的二进制文件，包含了运行应用程序所需的依赖项和配置信息，可以通过dockerfile来创建镜像

**容器（container）**：一个运行中的镜像实例，包含了应用程序和其依赖项，可以使用Docker CLI或者Docker Compose来创建和管理容器。

**仓库（registry）**：一个存储和分发镜像的地方，DockerHub是Docker官方提供的仓库，可以在里面找到许多常用的镜像，也可以在里面上传和分享自己的镜像。 

## 1.2 Docker特性

**1、高度可移植性**：可以在任何支持Docker的平台上运行，容器可以在不同环境中快速、简单部署和移植。

**2、轻量级和高效**：非常轻量级，启动和停止速度块，对系统资源的占用很少。以前一台服务器只能跑10台虚拟机，现在可以跑100个docker容器。

**3、隔离性好**：docker使用namespace和cgroups技术实现容器的隔离，每个容器拥有自己的进程、网络、文件系统和其他资源，保障了容器之间的隔离性和安全性。

**4、易于管理**：拥有一系列的命令行工具或图形化界面，方便管理和监控容器，还支持容器编排工具，入kubernetes，可以轻松管理和扩展容器集群。

**5、生态系统丰富**：Docker拥有庞大的社区和生态系统，提供了丰富的镜像和插件，可以方便的构建、部署和运行应用程序。



