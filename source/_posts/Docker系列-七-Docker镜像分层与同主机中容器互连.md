---
title: Docker系列(七)-Docker镜像分层与同主机中容器互连
date: 2021-07-05 15:28:40
categories:
- Docker
tags:
- Docker
- 学习笔记
---

## 一、Docker镜像分层

>镜像分层的好处：
>
>​	复用、节省磁盘空间，相同的内容只需加载一份到内存
>
>​	修改dockerfile后，重新构建时可以用缓存，速度快

### 1 查看docker镜像分层

通过导入镜像可以查看到镜像分层

```shell
docker load -i [镜像文件]
```

通过查看镜像历史可以查看到分层

```shell
[root@docker01 ~]# docker image history centos6.9_ssh:v2
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
57761235f898   2 days ago    /bin/sh -c #(nop)  CMD ["/usr/sbin/sshd" "-D…   0B
c0a2c21457c4   2 days ago    /bin/sh -c echo '123456' | passwd --stdin ro…   537B
29d10ff8b8e0   2 days ago    /bin/sh -c service sshd restart                 4.91kB
666b9ddfff15   2 days ago    /bin/sh -c yum install openssh-server -y        154MB
5b00553af9fc   2 days ago    /bin/sh -c #(nop) ADD file:65a30e1b327fec80b…   1.18kB
2199b8eb8390   2 years ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      2 years ago   /bin/sh -c #(nop)  LABEL name=CentOS Base Im…   0B
<missing>      2 years ago   /bin/sh -c #(nop) ADD file:0e6d175401c5b4260…   195MB
```

所有的这些层都会在`Docker`主机本地存储区域内存储，可以通过以下指令来列出：

```shell
ls /var/lib/docker/overlay2/
```

### 2  通过Dockerfile优化分层信息

- 尽量合并RUN和ADD来减少镜像分层数
- 新加的Dockerfile语句加到最后，不要加到前面

## 二、同主机中容器互连（--link是单向的）

>docker官方已不推荐使用docker run --link来链接2个容器互相通信，随后的版本中会删除--link

### 1 功能介绍

docker run --link可以用来链接2个容器，使得源容器（被链接的容器）和接受容器（主动去链接的容器）之间可以互相通信，并且接收容器可以获取源容器的一些数据，如源容器的环境变量。使用案例如下：

源容器启动：

```shell
docker run -d --name src_docker nginx 
容器ID:xxxx01, IP:172.16.0.2
```

接受容器连接：

```shell
docker run -d --name dest_docker --link src_docker:web centos7.9:v2
容器ID:xxxx02, IP:172.16.0.3
```

进入接受容器测试，不需要ping IP，直接ping别名就可以，web和src_docker都指向172.16.0.2<font color=red>（单向）</font>

```shell
docker exec -it xxxx01 /bin/bash
ping web
ping src_docker
```

> 接受容器的/etc/hosts将更新

### 2 案例：构建zabbix-server

启动一个mysql的容器

```shell
docker run --name mysql-server -t \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      -d mysql:5.7 \
      --character-set-server=utf8 --collation-server=utf8_bin 
```

启动java-gateway容器监控java服务

```shell
docker run --name zabbix-java-gateway -t \
      -d zabbix/zabbix-java-gateway:latest
```

启动zabbix-mysql容器使用link连接mysql与java-gateway

```shell
docker run --name zabbix-server-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
      --link mysql-server:mysql \
      --link zabbix-java-gateway:zabbix-java-gateway \
      -p 10051:10051 \
      -d zabbix/zabbix-server-mysql:latest
```

启动zabbix web显示，使用link连接zabbix-mysql与mysql

>zabbix的默认端口已有80改为8080，可见配置文件/etc/zabbix/nginx.conf

```she
docker run --name zabbix-web-nginx-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="zabbix_pwd" \
      -e MYSQL_ROOT_PASSWORD="root_pwd" \
      --link mysql-server:mysql \
      --link zabbix-server-mysql:zabbix-server \
      -p 8082:8080 \     
      -d zabbix/zabbix-web-nginx-mysql:latest
```

登录Zabbix

```she
浏览器访问：10.0.0.11:8082
Admin
zabbix
```

添加被监控节点-安装zabbix-agent

>获取zabbix-agent：
>
>uname -a 查看内核版本
>
>web页面查看zabbix版本
>
>https://www.zabbix.com/download 获取对应agent的安装方法

```shell
 rpm -Uvh https://repo.zabbix.com/zabbix/5.4/rhel/7/x86_64/zabbix-release-5.4-1.el7.noarch.rpm
 yum clean all
 yum install zabbix-agent
```

添加被监控节点-agent配置文件修改

>117行：Server=10.0.0.11, 注意防火墙和selinux的阻挡

```shell
vim /etc/zabbix/zabbix_agentd.conf
systemctl restart zabbix-agent
```

## 三、docker重启后容器不退出

> 默认情况下，systemctl restart docker之后，容器将处于Exited状态

### 1 添加容器启动参数

>docker重启后，容器先停止，再立即重新启动

```shell
docker run --restart=always
```

### 2 daemon配置文件修改（不推荐）

>docker重启后，容器不会停止，一直在运行，不推荐使用，不好控制

```shell
vim /etc/docker/daemon.json
# 添加一行，上行后面加逗号
"live-restore":true
```





​	



