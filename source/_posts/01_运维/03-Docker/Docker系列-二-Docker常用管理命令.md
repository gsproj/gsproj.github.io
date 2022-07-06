---
title: Docker系列(二)-Docker常用管理命令
date: 2021-06-28 22:59:33
categories:
- 运维
- （三）Docker
tags:
---

## 一、常用镜像管理命令

>​	Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数(如匿名卷、环境变量、用户等)。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

### 1 在镜像仓库查找镜像

```shell
docker search tomcat
```

### 2 在镜像仓库拉取镜像

>不指定版本号时默认下载最新版（latest），版本可在dockerhub(官方仓库)、DaoCloud(私有仓库)等仓库查到

```shell
# dockerhub拉取
docker pull alpine:3.6
# daocloud拉取
docker pull daocloud.io/jermine/alpine:latest
```

### 3 查看已有镜像

```shell
docker image ls
# 别名
docker images
```

### 4 导出镜像

>弃用export，导出的镜像不带版本TAG信息

```shell
docker image save alpine -o alpine.tar.gz
```

### 5 删除镜像

```shell
# 删除alpine镜像
docker image rm d4ff818577bc
```

### 6 导入镜像

>弃用import，导入的镜像不带版本TAG信息

```shell
docker image load -i alpine.tar.gz
```

### 7 查看镜像属性

```shell
docker image inspect 4f380adfc10f
```

### 8 镜像批量删除

```shell
docker image prune
```

### 9 指定TAG信息

>docker images查看docker image import的镜像，没有镜像名和TAG，可以使用此方法来修改

```sh
docker image tag d4ff818577bc oldbly
```

## 二、常用容器管理命令

>​	镜像(image)和容器(container)的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
>​	容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的root文件系统、自己的网络配置、自己的进程空间，甚至自己的用户ID空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立宿主的系统下操作一样。这种特性使容器封装的应用比直接在宿主运行更加安全。

### 1 运行容器

>1、docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态（必须夯住），否则这个容器，就会处于退出状态。
>
>2、业务在容器中运行：前台运行夯住，启动服务
>
>3、如果不指定执行命令，会运行默认的执行命令

```shell
# 放后台运行
docker run -d -p 80:80 nginx:latest
run 创建并运行一个容器
-d	放在后台运行
-p 	端口映射
-v 	源地址(宿主机)：目标地址(容器)

# 交互式方式进入容器执行
docker run -it --name centos6 centos:6.9 /bin/bash
-it 	分配交互式的终端
--name 	制定容器的名称
/bin/sh 容器执行的命令，每个进程默认有初始执行命令，可以覆盖
```

### 2 查看已有容器

>-a 显示所有容器（默认只显示running的容器）
>
>-l 显示最新的容器
>
>--no-trunc 显示完整id
>
>-q 静默输出（只显示容器id）

```shell
docker container ls -a
# 别名
docker ps -a
```

### 3 停止容器

```shell
docker container stop 55e9c7c
```

### 4 杀死容器

```shell
docker container kill 55e9c7c
```

>kill与stop的区别：
>
>- kill：不管容器同不同意，发送SIGKILL信号，强行终止。
>
>- stop：首先给容器发送一个SIGTERM信号，让容器做一些退出前必须的保护性、安全性操作，然后让容器自动停止运行，如果在一段时间内，容器还是没有停止，再发送SIGKILL信号，强行终止。

### 5 启动容器

```shell
docker container start 55e9c7c
```

### 6 进入容器（重要！调试、排错）

使用同一终端：

```shell
# 交互式方式运行容器（开启新终端）
docker run -it --name centos6.9 centos:6.9 /bin/bash
# 暂时退出当前终端
ctrl + p 再 ctrl + q
# 重新进入该终端
docker attach d4ff818577bc
```

使用不同终端（常用）

```shell
docker exec -it d4ff818577bc /bin/bash
```

### 7 删除容器

```shell
docker container rm 55e9c7cb59a6 55e9c7cb59a5
# 别名
docker rm 55e9c7cb59a6
# 批量删除容器
docker rm `docker ps -a -q`
```

