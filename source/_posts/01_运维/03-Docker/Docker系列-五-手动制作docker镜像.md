---
title: Docker系列(五)-手动制作docker镜像
date: 2021-06-29 13:35:26
categories:
- 运维
- （三）Docker
tags:
---

## 一、制作Docker镜像

### 1 启动基础容器

```she
docker run -it centos:6.9 # yum
docker run -it alpine:3.9 # apk
```

### 2 在容器中安装服务

修改yum源（Centos6阿里源已停止维护）

```shell
echo '[centos-office]
name=centos-office
failovermethod=priority
baseurl=https://vault.centos.org/6.10/os/x86_64/
gpgcheck=1
gpgkey=https://vault.centos.org/6.10/os/x86_64/RPM-GPG-KEY-CentOS-6' > CentOS-Base.repo
```

安装并启动openssh服务

```shell
yum install openssh-server -y
service sshd restart
```

修改root密码(默认没有密码)

```shell
echo '123456' | passwd --stdin root
# 或者
echo root:123456 | chpassw
```

将已经安装好sshd服务的容器打包成镜像

```shell
docker container commit 981877f137c9 centos6.9_ssh:v1
```

测试镜像

>sshd -D：以后台守护进程的方式运行服务

```shell
# 启动sshd并将22端口映射出来，可以使用xshell连接
docker run -d -p 1022:22 centos6.9_ssh:v1 /usr/sbin/sshd -D
```

## 二、小案例：创建一个ssh+nginx双服务的镜像

创建容器

```shell
docker run -d -p 1023:22 centos6.9_ssh:v1 /usr/sbin/sshd -D
```

修改yum源和epel源

```she
由于Centos6阿里云停止维护
参考：https://blog.csdn.net/u013250554/article/details/110684307
```

安装nginx

```shell
yum install nginx -y
```

创建运行服务的脚本

```shell
vim /root/init.sh
#!/bin/bash
service sshd restart
nginx -g 'daemon off;'
```

将容器封装成镜像

```shell
docker commit e6a6dsa6 centos6.9_ssh_nginx:v2
```

启动镜像，开启服务，并夯住

>可以使用工具ssh登录，并且可以访问到nginx的欢迎页面

```shell
docker run -d -p 1025:22 -p 80:80 centos6.9_ssh_nginx:v2 /bin/bash /root/init.sh
```

## 三、通过环境变量设置容器密码

修改/root/init.sh文件

```shell
#!/bin/bash

if [ -z $SSH_PWD ];then
        SSH_PWD=123456
fi
echo "$SSH_PWD" | passwd --stdin root

service sshd restart
nginx -g 'daemon off;'
```

打包成镜像

```shell
docker commit 12386c6504d4 centos6.9_ssh_nginx_passwd
```

运行容器

>docker run -e：指定环境变量

```shell
docker run -d -p 1022:22 -p 80:80 -e "SSH_PWD=123456" centos6.9_ssh_nginx_passwd /bin/bash /root/init.sh
```

