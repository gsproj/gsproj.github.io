---
title: Docker系列(一)-安装Docker
date: 2021-06-28 16:06:37
categories:
- 运维
- （三）Docker
tags:
---

## 一、容器简介

>容器就是在隔离环境中运行一个进程，如果进程停止，容器就会销毁，隔离环境拥有自己的系统文件，ip地址，主机名等。

### 1 容器和虚拟化的区别

- KVM虚拟化：

  - 需要硬件的支持，需要模拟硬件，可以运行不同的操作系统，启动时间分钟级（有开机启动流程）

    开机启动流程

    bios开机硬件自检

    根据bios设置的优先启动项boot

    读取mbr/gpt引导，读取mbr硬盘分区信息，内核加载路径

    加载内核

    启动第一个进程（C6：/sbin/init，C7：systemd）

    系统初始化完成

    运行服务

- 容器：

  - 不需要硬件的支持，不需要模拟硬件，公用宿主机内核，启动时间秒级（没有开机启动流程）

    容器的第一个进程直接运行服务，损耗少，启动快，性能高

### 2 容器的优缺点

- 优点

  - 与宿主机使用同一个内核，性能损耗小

    不需要指令级模拟

    容器可以再cpu核心的本地运行指令，不需要任何专门的解释机制

    避免了准虚拟化和系统调用替换中的复杂性

    轻量级隔离，在隔离的同事还提供共享机制，以实现容器与宿主机的资源共享

- 缺点

  - 使用同一内核，存在安全性问题

### 3 容器技术的发展过程

> chroot --- lxc ---- docker

### 4 Docker组成

​	Docker基于Go语言开发，C/S模式

- 主要组件
  - 镜像
  - 容器
  - 仓库：最大的dockerhub
  - 网络
  - 存储

## 二、Docker安装

>参考网站：https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

### 1 联网在线安装

开启rpm包缓存，方便制作离线安装包

```shell
vim /etc/yum.conf
keepcache=1 
```

如果你之前安装过 docker，请先删掉

```shell
sudo yum remove docker docker-common docker-selinux docker-engine
```

安装一些依赖

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

根据你的发行版下载repo文件（Centos）

```shell
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
```

把软件仓库替换为TUNA：

```shell
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

最后安装:

```shell
sudo yum makecache fast
    sudo yum install docker-ce
```

### 2 无网环境下离线安装

搜集联网环境下下载的rpm包

```shell
find /var/cache/yum/x86_64/7/ -name "*.rpm" | xargs -i mv {} docker_rpm/
tar -zvcf docker_rpm.tgz docker_rpm/
```

拷贝到无网环境的服务器中安装

```shell
tar -vxf docker_rpm.tgz # 解压
cd docker_rpm	
rpm -Uvh ./*.rpm # 安装
```

### 3 启动服务并验证安装是否成功

```shell
# 启动服务
systemctl enable docker
systemctl start docker

# 验证是否安装成功
docker version
docker info
```

### 4 Docker镜像下载加速

- 阿里云docker镜像加速器服务
- 配置docker镜像加速(推荐)

```shell
# 创建文件
vi /etc/docker/daemon.json
{    
	"registry-mirrors":["https://registry.docker-cn.com"]
} 
# 重新加载
systemctl daemon-reload
systemctl restart docker
```

### 5 创建并运行一个nginx容器

```shell
docker run -d -p 80:80 nginx
```

