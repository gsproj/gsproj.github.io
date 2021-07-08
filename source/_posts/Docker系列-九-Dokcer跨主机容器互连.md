---
title: Docker系列(九)-Dokcer跨主机容器互连
date: 2021-07-06 21:21:33
categories:
- Docker
tags:
- Docker
- 学习笔记
---

## 一、Docker容器的四种网络类型

>哪四种？
>
>- bridge（默认）：NAT桥接模式
>- none：不分配网络，什么服务都访问不了
>- host：与宿主机共享网络，共享主机名，端口共用(宿主机用了的端口，容器也不能用)，【网络性能最高】
>- container:容器id：与容器xx共享网络，共享主机名、hosts、hostname、端口等....【K8S常用】

### 1 指定与查看容器网络类型的方法

指定容器网络类型

```shell
docker run --network none # 指定网络类型
```

查看容器网络类型

```shell
docker instpect 容器id # 可以查看当前的容器网络类型
# 查看字段
"NetworkSettings": {
	"Bridge": "",
```

查看有哪些网络类型

```shell
docker network ls # 查看有哪些网络类型
```



## 二、使用macvlan实现

>优势：
>
>​	性能比overlay高
>
>​	不用做端口映射，外界可直接访问
>
>劣势：
>
>​	IP需要手动指定

### 1 案例：使用macvlan实现两个centos6.9_ssh容器跨主机网络通信

>宿主机信息（虚拟机）：
>
>​	docker01: 10.0.0.11 网关: 10.0.0.2
>
>​	docker02: 10.0.0.12 网关: 10.0.0.2

宿主机1,2分别创建macvlan

```shell
docker network create --driver macvlan --subnet 10.0.0.0/24 --gateway 10.0.0.2 -o parent=ens33 macvlan_1
```

设置网卡为混杂模式【Ubuntu需要设置】

>混杂模式是计算机网络中的术语。 是指一台机器的网卡能够接收所有经过它的数据流，而不论其目的地址是否是它。

```shell
ip link set ens33 promisc on
```

宿主机1,2分别使用centos7.9_ssh:v2镜像创建容器，并指定为macvlan_1网络

```shell
# docker01
docker run -d --network macvlan_1 --ip=10.0.0.100 centos6.9_ssh:v2
# docker02
docker run -d --network macvlan_1 --ip=10.0.0.200 centos6.9_ssh:v2
```

测试，docker exec进入docker01中运行容器，开启抓包，并使用docker02中的容器ping它

```shell
tcpdump icmp
```

通过xshell或者mobaxterm可以直接ssh到容器中 (PS:并没有 -p 22端口)

> 实际测试宿主机并不能ssh到容器，显示No route to host，但是物理机可以连接

```shell
ssh root@10.0.0.100
```



## 三、使用overlay实现

>优势：
>
>​	可以自动分配ip地址
>
>劣势：
>
>​	需要做端口映射才能访问容器服务
>
>overlay参考：https://www.cnblogs.com/CloudMan6/p/7270551.html

### 1 案例：使用overlay实现两个centos6.9_ssh容器跨主机网络通信

docker01启动consul

>consul是一个key:value类型的存储数据库

```shell
docker run -d -p 8500:8500 --restart=always -h consul --name consul progrium/consul -server -bootstrap
```

docker01,02上设置daemon.json文件

```shell
vim /etc/docker/daemon.json
```

```json
"hosts":["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
"cluster-store":"consul://10.0.0.11:8500",
"cluster-advertise":"10.0.0.11:2376" # 此处不同，docker01为10.0.0.11，docker02为12
```

修改docker.service文件

>因为daemon.json中的hosts项与docker.service中的-H参数冲突，需要去掉

```shell
vim /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# 删除-H fd://
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock
```

重启docker服务

```shell
systemctl daemon-reload
systemctl restart docker
```

测试consul是否搭建成功

```shell
浏览器访问：http://10.0.0.11:8500/
在KEY/VALUE标签页正常显示10.0.0.11和12两台宿主机
```

docker01,02创建overlay网络

```shell
docker network create -d overlay --subnet 172.16.1.0/24 --gateway 172.16.1.254 ol1
```

启动容器

```shell
# docker01
docker run -d --network ol1 --name centos6.9_01 centos6.9_ssh:v2
# docker02
docker run -d --network ol1 --name centos6.9_02 centos6.9_ssh:v2
```

测试容器间网络

```shell
# docker01中的容器ping另一容器的hostname
[root@e1e8cc01792d /]# ping centos6.9_02
PING centos6.9_02 (172.16.1.2) 56(84) bytes of data.
64 bytes from centos6.9_02.ol1 (172.16.1.2): icmp_seq=1 ttl=64 time=0.197 ms
64 bytes from centos6.9_02.ol1 (172.16.1.2): icmp_seq=2 ttl=64 time=0.269 ms
64 bytes from centos6.9_02.ol1 (172.16.1.2): icmp_seq=3 ttl=64 time=1.04 ms
```

### 2 Overlay的网络访问流程图
{% asset_img overlay网络访问.png overlay网络访问 %}

