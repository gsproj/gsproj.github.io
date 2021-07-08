---
title: Docker系列(十)-Dokcer单机编排docker-compose
date: 2021-07-07 09:51:12
categories:
- Docker
tags:
- Docker
- 学习笔记
---

>docker-compose 单机版的容器编排工具

## 一、安装docker-compose

添加Centos7的epel源

```shell
curl -o epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
```

安装pip

>Python 2.7已于2020年1月1日到期，请停止使用。请升级您的Python，因为不再维护Python 2.7。pip 21.0将于2021年1月停止对Python 2.7的支持。pip 21.0将删除对此功能的支持。因此安装<21.0的版本

```she
yum install -y python2-pip
pip install --upgrade "pip < 21.0"
```

安装docker-compose

```shell
pip install docker-compose
```

创建文件夹用于存放docker-compose脚本

```shell
mkdir /opt/docker-compose
```

## 一、案例：compose构建wordpress并使用nginx负载均衡

创建docker-compose文件

```shell
vim /opt/docker-compose/wordpress/docker-compose.yml
```



## 二、案例：构建zabbix