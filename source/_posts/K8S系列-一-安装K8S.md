---
layout: w
title: K8S系列(一)-安装K8S
date: 2021-07-08 23:05:17
tags:
---

## 一、K8S介绍

> 全称Kubernetes，是一个Docker管理平台

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

## 三、K8S使用

1 查看节点

```shell
kubectl get nodes
```

2 查看服务状态

```shell
kubectl get componentstatus
```

