---
title: Docker系列(零)-Docker介绍
date: 2021-06-28 16:06:37
categories:
- 运维
- （三）Docker
tags:
---

# 一、容器简介

容器就是在隔离环境中运行一个进程，如果进程停止，容器就会销毁。

隔离环境拥有自己的系统文件，ip地址，主机名等。

## 1.1 容器和虚拟化的区别

KVM虚拟化：

需要硬件的支持，需要模拟硬件，可以运行不同的操作系统，启动时间分钟级（有开机启动流程）

开机启动流程

bios开机硬件自检

根据bios设置的优先启动项boot

读取mbr/gpt引导，读取mbr硬盘分区信息，内核加载路径

加载内核

启动第一个进程（C6：/sbin/init，C7：systemd）

系统初始化完成

运行服务

容器：

不需要硬件的支持，不需要模拟硬件，公用宿主机内核，启动时间秒级（没有开机启动流程）

容器的第一个进程直接运行服务，损耗少，启动快，性能高

 

## 1.2 容器的优缺点：

优点：

与宿主机使用同一个内核，性能损耗小

不需要指令级模拟

容器可以再cpu核心的本地运行指令，不需要任何专门的解释机制

避免了准虚拟化和系统调用替换中的复杂性

轻量级隔离，在隔离的同事还提供共享机制，以实现容器与宿主机的资源共享

缺点：

使用同一内核，存在安全性问题

 

## 1.3 容器技术的发展过程

chroot --- lxc ---- docker

 

# 二、Docker安装

\# 添加docker安装源 

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 

查看所有仓库中docker版本，并选择特定版本安装：(此处我们查看社区版 docker-ce) yum list docker-ce --showduplicates | sort -r # 安装docker-ce yum install docker-ce -y

 

# 三、Docker镜像的常用命令

\# 搜索镜像docker search [镜像名称]# 拉取镜像docker pull [域名镜像名称]:[版本号]例如：docker pull daocloud.io/hzc/alpine:3.6 # 默认拉取Lastest最新版PS: docker image pull = docker pull# 如何查看镜像有那些版本？dockerhub网页搜索 daocloud 国内的dockhub# 镜像加速：（1）阿里云docker镜像加速器服务（2）配置docker镜像加速(推荐) vi /etc/docker/daemon.json {   "registry-mirrors":[“https://registry.docker-cn.com”] } systemctl daemon-reload# 上传镜像docker push [镜像名称]# 查看已有镜像docker images = docker image ls# 导出镜像docker image save alpine:latest -o docker_alpine.tar.gz # save跟export选哪个？都是导出镜像，但是export没带版本标签，export弃用# 导入镜像docker image load -i docker_alpine.tar.gz # load跟import选哪个？都是导入镜像，load对应save，import不带版本标签，import弃用# 删除镜像docker image rm alpine:3.6# 构建镜像docker image build# 查看构建镜像用到的历史命令docker image histroy# 查看镜像的详细属性docker image inspect# 批量删除镜像docker image prune# 给镜像打标签docker image tag [镜像ID] oldboy:v1 # Docker的容器管理1、查看容器列表 docker container ls -a docker ps # 默认只查看活着的容器 docker ps -a # 查看所有容器 docker ps -a -q # 静默输出，显示所有容器的ID docker ps -a -l docker ps -a -l --no-trunc # 查看完整命令（不隐藏）# 停止容器docker container stop [容器ID] docker container kill [容器ID]# 恢复容器 docker container start 【容器ID】# 启动容器 docker run -d -p 80:80 nginx:latest run 创建并运行一个容器 -d   放在后台运行 -p 端口映射 -v 源地址(宿主机)：目标地址(容器) docker run -it --name centos6 centos:6.9 /bin/bash -it 分配交互式的终端 --name 制定容器的名称 /bin/sh 容器执行的命令，每个进程默认有初始执行命令，可以覆盖※进入容器（调试、排错）docker exec - it [容器名称/ID] /bin/bash docker attach 【容器名称/ID】 (使用同一个终端)临时退出容器：ctrl +p和ctrl + q退出# 删除容器docker container rm [容器ID] docker rm [容器ID]如何批量删除容器：docker rm `docker ps -a -q`

 

**总结：**

docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态（必须夯住），否则这个容器，就会处于退出状态。

业务在容器中运行：前台运行夯住，启动服务

 

# 四、 Docker的网络访问

## 4.1 容器网络访问流程

实际上是端口映射，docker容器有自己的ip，需要靠宿主机NAT上网 

 

 

 -p设置自动端口映射，在iptables中有增的Chain Docker, 也可以手动设置NAT

查看当前设置的nat：iptables -t nat -L -n

 

## 4.2 容器网络访问注意事项：

sysctl -a | grep ipv4 | grep forward 

查看

net.ipv4.ip_forward = 1 # 为1时，docker容器才能上网，虚拟机挂起将使他变成0

解决方法:

1、sysctl net.ipv4.ip_forward = 1 设置为1

2、不要挂起虚拟机，直接关机重启，docker服务在启动时会将它改为1

 

## 4.3 指定映射(-p)参数详解

-p hostPort:containerPort # 指定端口 -p ip:hostPort:containerPort # 指定ip+端口 -p ip::containerPort # 指定随机端口 -p 10.0.0.100:53:udp # 指定随机端口 + udp -p hostPort:containerPort -p hostPort:containerPort # 指定多个端口

# 五、容器的数据卷挂载

## 5.1 临时挂载

\# 将/opt/xiaoniao目录挂载到容器的html目录 docker run -d -p 80:80 -v /opt/xiaoniao:/usr/share/nginx/html nginx:latest

## 5.2 持久化挂载

容器被删除，创建的卷可以保留，可以再次挂载到新建的容器中

\# 创建名为oldboy的容器卷并挂载到容器html目录 docker run -d -p 80:80 -v oldboy:/usr/share/nginx/html nginx:latest # 查看当前有哪些容器卷 docker volume ls # 查看名为oldboy的卷的信息 docker volume inspect oldboy # PS：删除容器并删除卷，无法将卷删除 docker rm -f -v [容器ID]   -v --volume

# 六、小案例练习

\>> 基于Nginx多端点的多站点 基于nginx启动一个容器，监听80和81，访问80，出现nginx默认的欢迎首页，访问81，出现小鸟页面。

# 七、如何制作Docker镜像

## 7.1 启动一个基础的容器

docker run -it centos:6.9 # yum docker run -it alpine:3.9 # apk

## 7.2 容器中安装服务

\# dns重定向 echo '192.168.15.84 mirrors.aliyun.com' >> /etc/hosts # 替换源为阿里源 curl -o /etc/yum.repo.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/CentOS-6.repo # 安装并启动openssh服务 yum install openssh-server -y service opensshd restart # 修改root密码 echo '123456' | passwd --stdin root 或者 echo 123456:root | chpassw

7.3 把已经安装服务的容器打包成镜像

docker contanier commit 5617e5d123432 centos6.9_ssh:v1

## 7.4 测试镜像的功能

\# 使用镜像启动一个新容器,并开启ssh服务 docker run -d -p 1022:22 centos6.9_ssh:v1 tail -f /usr/sbin/sshd -D

## 7.5 创建一个ssh + nginx双服务的镜像

(1) 启动一个基础容器 docker yun -it -p 80:80 -p 1023:22 centos6.9_ssh:v1 /bin/bash (2) 在容器中安装服务(hosts与repo源在新容器会重新挂载) # dns重定向 echo '192.168.15.84 mirrors.aliyun.com' >> /etc/hosts # 替换源为阿里源 curl -o /etc/yum.repo.d/epel.repo https://mirrors.aliyun.com/repo/epel.repo # 安装nginx服务 yum install nginx -y (3) 把已经安装好服务的容器，提交为镜像 docker commit e6a6dsa6 centos6.9_ssh_nginx:v2 (4) 测试镜像功能 vim init.sh >>>>> #!/bin/bash service sshd restart nginx -g 'daemon off;' >>>>> # 启动镜像，执行脚本：开启服务，并夯住 docker run -d -p 1025:22 -p 80:80 centos6.9_ssh_nginx:v2 /bin/bash /init.sh

### 7.6 自定义容器镜像的密码

\# 修改脚本，添加密码相关的脚本行，见右图 vim init.sh # 启动容器，并附带环境变量 docker run -d -p 1025:22 -p 80:80 -e "SSH_PWD=123456" centos6.9_ssh_nginx:v2 /bin/bash /init.sh

 

 

###  

 

 

### 7.7 作业

制作基于centos6的lnmp架构的镜像，discuz论坛

怎么夯住？

启动所有需要的服务

最后tail -F (大F无论文件有没有) 

#  

# 八、Dockfile的使用

发布镜像太大了，而dockerfile只有几kb，使用dockfile文件可以构建出相同的镜像，

## 8.1 使用dockfile自动构建镜像

自动构建镜像的步骤：

1、手动构建一遍

2、参考历史命令，编写dockerfile

3、构建镜像

dockerfile build -t centos6.9_ssh .

4、测试

## 8.2 Dockerfile常用命令详解

 

 
