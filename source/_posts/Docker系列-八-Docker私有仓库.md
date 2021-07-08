---
title: Docker系列(八)-Docker私有仓库
date: 2021-07-06 13:58:52
categories:
- Docker
tags:
- Docker
- 学习
---

## 一、官方私有仓库registry

拉取私有仓库镜像

```shell
docker pull registry
```

启动私有仓库容器

```shell
docker run -di --name=registry -p 5000:5000 registry
```

验证是否正常

```shell
# 浏览器输入
10.0.0.12:5000/v2/_catalog
```

修改daemon.json，让 docker信任私有仓库地址

```shell
vi /etc/docker/daemon.json
# 添加
{
	"insecure-registries":["10.0.0.12:5000"]
} 
```

重启docker服务

```shell
systemctl reset-failed docker.service
```

上传镜像到私有仓库

```shell
docker push 10.0.0.11:5000/registry
```

从私有仓库下载镜像

```shell
docker pull http://10.0.0.12:5000/registry
```

## 二、企业级私有仓库Harbor

>为什么通harbor？因为官方私有仓库：
>
>- 网页简陋，查看镜像、删除镜像不方便
>- 权限控制不方便（要么有权限，要么完全没权限）

