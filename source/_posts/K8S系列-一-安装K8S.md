---
layout: w
title: K8S系列(一)-安装K8S
date: 2021-07-08 23:05:17
tag:
---

课程目录：

>第一部分：K8S概念和架构
>
>第二部分：K8S安装
>
>​	kubeadm
>
>​	二进制
>
>第三部分：K8S核心概念
>
>POD 
>
>CONTROLLER
>
>SERVICE
>
>INGRESS
>
>RABC
>
>HELM
>
>持久化存储
>
>第四部分：集群监控平台
>
>第五部分：从零开始搭建高可用K8S集群
>
>第六部分：在集群环境中部署项目
>
>

## 一、K8S概念和架构

### 1 K8S概述和特性

#### 1.1 基本介绍

- K8S是谷歌在2014年开发的容器化集群管理系统
- 使用K8S可以进行容器化应用部署
- 使用K8S利于应用扩展
- K8S目标是让部署容器化应用更加简洁和高效

容器化部署的好处：

#### 1.2 K8S的特性和优势

自动装箱：自动部署应用容器

自我修复（自愈能力）： 

水平扩展：副本数量增加

服务发现（负载均衡）：通过Service实现，为多个副本对外提供统一的入口，节点调度负载均衡

滚动更新：

版本回退：

密钥配置管理：不需要重新构建镜像，可以部署和更新密钥和应用配置

存储编排：

批处理：	

### 2 K8S架构组件

### 3 K8S核心概念

3.1 Pod

3.2 Controller

3.3 Service



## 二、K8S安装

>master  192.168.44.146
>
>node1	192.168.44.145
>
>node2	192.168.44.144

### 1 kubeadm安装



### 2 二进制安装



-----------------------------------------------------------------------

生产环境通用需求：

​	服务的自动发现和负载均衡

​	自愈

​	一键升级和回滚

​	水平扩展（弹性伸缩） 

### 1 容器管理平台

docker swarm

messos

marathon

kubernetes  (90%市场)

## 2 K8S发展

发布频繁，一年4个版本	

### 3 核心组件



## 二、K8S安装

>K8S安装方式很多：
>
>- 源码编译安装 ：golang编译环境
>- 二进制安装 ：文档，全程手动，ansible等
>- kubeadm安装：网络要求
>- minikube ：开发者学习
>- yum安装 
>
>这里使用yum安装的方法

### 1 虚拟机准备

> centos7, 1CPU 1G
>
> 设置主机名，**添加hosts解析**

```shell
k8s-master  --  10.0.0.21
k8s-node1   --  10.0.0.22
k8s-node2   --  10.0.0.23
```

配置Centos7源和epel7源

```he
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
```

### 2 Master节点安装

#### 2.1 安装docker

```shell
yum install docker -y
```

#### 2.2 安装etcd

>etcd是一个nosql数据库

```shell
yum install etcd -y
```

修改etcd配置文件

```shell
vim /etc/etcd/etcd.conf
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.0.21:2379"
```

启动etcd服务

```shell
systemctl enable etcd
systemctl restart etcd
```

测试etcd

```shell
etcdctl set testdir/testkey0 0
etcdctl get testdir/testkey0
```

etcdctl相关设置

```shell
etcdctl -C http://10.0.0.21:2379 cluster-health
```

#### 2.3 安装k8s服务

```shell
yum install kubernetes-master -y
```

配置文件修改

```shell
vim /etc/kubernetes/apiserver
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBE_ETCD_SERVERS="--etcd-servers=http://10.0.0.21:2379"
# 23行删除ServiceAccount，如下
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota" # 
```

```shell
vim /etc/kubernetes/config
KUBE_MASTER="--master=http://10.0.0.21:8080"
```

开放防火墙端口【重要】

```shell
firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

启动服务

```shell
systemctl enable kube-apiserver
systemctl restart kube-apiserver
systemctl enable kube-controller-manager
systemctl restart kube-controller-manager
systemctl enable kube-scheduler
systemctl restart kube-scheduler
```

### 3 Node节点安装

#### 3.1 安装k8s-node服务 

```shell
yum install kubernetes-node
```

编辑配置文件

```shell
vim /etc/kubernetes/config
KUBE_MASTER="--master=http://10.0.0.21:8080"
```

```shell
vim /etc/kubernetes/kubelet
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
KUBELET_HOSTNAME="--hostname-override=10.0.0.22"
KUBELET_API_SERVER="--api-servers=http://10.0.0.21:8080"
```

启动服务

```shell
systemctl enable kubelet
systemctl restart kubelet
systemctl enable kube-proxy
systemctl restart kube-proxy
```

#### 3.2 fannel网络服务安装

>用于节点之间通信

所有节点安装并配置fannel

```shell
yum install flannel -y
```

```shell
vim /etc/sysconfig/flanneld
FLANNEL_ETCD_ENDPOINTS="http://10.0.0.21:2379"
```

```shell
etcdctl mk /atomic.io/network/config '{"Network":"172.16.0.0/16"}'
```

重新启动服务---Master节点

```shell
systemctl enable fanneld
systemctl restart fanneld
systemctl restart kube-apiserver
systemctl restart kube-controller-manager
systemctl restart kube-scheduler	
```

重新启动服务---Node节点

```shell
systemctl enable fanneld
systemctl restart fanneld
systemctl restart kubelet
systemctl restart kube-proxy
```

### 4 配置Master为镜像服务器

## 三、K8S使用

1 查看节点

```shell
kubectl get nodes
```

2 查看服务状态

```shell
kubectl get componentstatus
```

### 1 Pod使用

创建pod

```shell
kubectl create -f k8s_pod.yml
```

删除pod

```she
kubectl delete pod oldboy
kubectl delete pod --all 删除所有
```

查看pod

```shell
kubectl get pods
kubectl get pods -o wides
```

查看pod详细信息

```shell
kubectl descripe pod nginx
```

 更新

```shell
kubectl replace xxxx.yml
```



#### 1.1 创建一个pod,为什么要启动两个容器？

>一个pod中可以挂多个容器

比如创建一个niginx pod，将启动一个pod容器，一个nginx容器

Pod容器172.16.18.2

Nginx容器，共用pod容器ip

主要通过用POD来实现K8S的高级功能



#### 1.2 rc副本控制器的使用

通过rc保证容器高可用

调整rc副本数

```shell
kubectl scale rc myweb --replicates=1
```

#### 1.3 利用rc实现滚动升级和一键回滚

案例：nginx 1.13升级1.15

```shell
# 每5s升级一个
kubectl rolling-update myweb -f nginx-rc1.15.yaml --update-period=5s
```

案例：nginx1.15回滚到1.13

```shell
kubectl rolling-update mywebv2 -f nginx-rc1.13.yaml
```



### service的创建和访问

> 外部访问容器，端口映射

#### K8S小结

1 端口30000开始？

2 查看yml字段编写帮助

```shell
kubectl explain svc.spec
```

