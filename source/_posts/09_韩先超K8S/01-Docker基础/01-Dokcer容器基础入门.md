---
title: 01-Docker基础
date: 2024-4-24 16:30:52
categories:
- 超哥K8S
- （一）Docker容器基础入门
tags:
---

# 一、Docker介绍

## 1.1 Docker现状

K8S已废弃docker，改用containerd（轻量级的容器运行时）

还有一个Podman

## 1.2 Docker的优缺点

优点：

​	快：性能快、管理操作快

​	敏捷：像虚拟机一样敏捷，但更便宜

​	灵活：将应用和系统容器化，不需要额外的操作性系统

​	轻量：一台服务器上可以部署100~1000个Container容器

​	便宜：开源、免费

​		docker-ce：社区版

​		docker-ee：商业版

缺点：

​	所有容器共用linux内核资源，资源能否实现最大限度的利用，因此安全上存在漏洞。

​	物理机内核出问题，所有容器都会受影响。



# 二、环境准备

1、虚拟机配置：

| 名称    | 配置                     | IP             | 操作系统   |
| ------- | ------------------------ | -------------- | ---------- |
| Master1 | 2核2G、硬盘100G、NAT网络 | 192.168.40.180 | CentOS 7.7 |
|         |                          |                |            |
|         |                          |                |            |

2、服务器系统初始化操作

设置主机名、关闭firewalld、关闭iptables、关闭selinux

时间同步

```shell
yum install ntp ntpdate -y
ntpdate cn.pool.ntp.org
# 写到计划任务中，每一小时同步一次
crontab -e
*/1 * * * * /sbin/ntpdate cn.pool.ntp.org &>/dev/null
systemctl restart crond
```

安装常用软件

```shell
yum install -y wget net-tools nfs-utils lrzsz gcc gcc-c++ make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel vim ncurses-devel autoconf automake zlib-devel python-devel epel-release openssh-server socat ipvsadm conntrack yum-utils
```



3、安装Docker-ce

配置docker-ce国内的yum源（阿里云）

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装docker依赖包

```shell
yum install -y device-mapper-persistent-data lvm2
```

安装docker-ce

```shell
yum install docker-ce -y
```

启动docker服务器

```shell
systemctl start docker && systemctl enable docker
```



4、查看docker版本

```shell
docker verison

[root@master1 ~]#docker version
Client:
 Version:           18.09.9
 API version:       1.39
 Go version:        go1.11.13
...
```



5、开启包转发功能和修改内核参数

br_netfilter模块用于将桥接流量转发至iptables链，需要开启转发。

```shell
# 开启模块
modprobe br_netfilter

# 验证是否已加载
[root@master1 /etc/sysctl.d]#lsmod | grep br_netfilter
br_netfilter           22256  0 
bridge                151336  1 br_netfilter
```

创建配置文件，修改内核参数

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



