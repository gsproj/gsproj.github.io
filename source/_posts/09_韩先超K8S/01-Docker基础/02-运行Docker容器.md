---
title: 02-运行Docker容器
date: 2025-3-3 15:12:52
categories:
- 超哥K8S
- （一）Docker容器基础入门
tags:
---

# 一、Docker容器操作

## 1.1 交互式容器

1、创建交互式Docker容器

```shell
# 以交互式方式运行容器，并进入容器
docker run  --name=hello -it centos /bin/bash
```

![image-20250303151500931](./../../../img/image-20250303151500931.png)

2、查看正在运行的容器

```shell
docker ps
```

![image-20250303151630396](./../../../img/image-20250303151630396.png)

3、以守护进程方式运行容器

>在1.1中创建的hello容器比较脆弱，只要exit退出，容器也会停止，需要设置守护进程让容器一直运行。

以后台守护进程方式运行容器

```shell
docker run --name=hello -td centos /bin/bash
```

交互式进入容器

```shell
docker exec -it hello /bin/bash
```

## 1.2 查找容器

查看容器（包括已经停止删除的容器）

```shell
docker ps -a
```

## 1.3 停止/启动容器

1、停止容器

```shell
docker stop 【容器名 或者 容器ID】
# 实践
docker stop hello
# 或者
docker stop 926b
```

2、启动容器

```shell
docker stop 【容器名 或者 容器ID】
# 实践
docker start hello
# 或者
docker start 926b
```

## 1.4 删除容器

删除hello(926b)容器

```shell
# 正常删除
docker rm 926b
# 强制删除
docker rm 926b --force
```



