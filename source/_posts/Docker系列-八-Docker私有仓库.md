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

### 1 安装步骤

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

>Harbor：第三方registry组件

www.qstack.com.cn

### 1 为什么使用Harbor

因为官方仓库registry存在诸多问题：

- https问题

- 网页简陋，查看镜像、删除镜像不方便
- 权限控制不方便（要么有权限，要么完全没权限），不支持多用户

### 2 Harbor安装步骤（需验证一遍）

离线安装包获取

```shell
https://github.com/goharbor/harbor
```

修改配置文件

```shell
hostname = 10.0.0.12  
harbor_admin_pass = 1qaz@WSX
```

执行安装脚本（时间比较长）

```shell
./install.sh
```

>PS：common文件夹内的文件用于挂载

网页访问

```shell
http://10.0.0.12
admin
```

### 3 镜像推送与下载

docker配置文件添加白名单

```shell
vim /etc/docker/daemon.json
insecure-registry = "10.0.0.12" # 不要加端口,可以是IP或域名
```

镜像打标签

```shell
docker tag alpine:latest 10.0.0.12/xxx/alpine:latest   # xxx是仓库中的项目名
```

登录到仓库

```shell
docker login 10.0.0.12
```

推送到仓库

```shell
docker push 10.0.0.12/xxx/alpine:latest
```

从仓库下载镜像

```shell
docker pull 10.0.0.12/xxx/alpine:latest
```

### 4 将Harbor升级为https访问

配置文件修改

```shell
# 使用https协议
ui_url_protocol = https 
# 证书配置
ssl_cert = 指定crt文件的路径
ssl_cert_key = 指定key文件的路径
```

再次执行安装脚本

```shell
./install.sh
```

