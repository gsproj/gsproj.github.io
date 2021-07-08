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

### 1 步骤：yum安装K8S

虚拟机准备

```shell
centos7 1CPU 1G
k8s-master  --  10.0.0.11 
k8s-node1   --  10.0.0.12
k8s-node2   --  10.0.0.13
设置主机名，添加hosts解析
```

master节点安装etcd

>etcd是一个nosql数据库

```shell
yum install etcd -y
```

修改etcd配置文件



