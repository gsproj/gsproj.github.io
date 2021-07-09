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
>
>项目地址：https://github.com/goharbor/harbor
>
>老男孩强哥博客地址：https://oldqiang.com/

### 1 为什么使用Harbor

因为官方仓库registry存在诸多问题：

- https问题

- 网页简陋，查看镜像、删除镜像不方便
- 权限控制不方便（要么有权限，要么完全没权限），不支持多用户

### 2 Harbor安装步骤

>这里以v2.2.3为例

离线安装包获取

```shell
wget https://github.com/goharbor/harbor/releases/download/v2.2.3/harbor-offline-installer-v2.2.3.tgz
```

解压文件，并修改配置文件

```shell
tar -vxf harbor-offline-installer-v2.2.3.tgz
cd harbor/
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
# 注释https设置项，并修改以下内容
hostname = 10.0.0.12  
harbor_admin_password = 1qaz@WSX
```

执行安装脚本（时间比较长）

>需要先安装docker-compose，
>
>[docker-compose安装参考](https://gsproj.github.io/2021/07/07/Docker%E7%B3%BB%E5%88%97-%E5%8D%81-Dokcer%E5%8D%95%E6%9C%BA%E7%BC%96%E6%8E%92docker-compose/)

```shell
./install.sh
```

网页访问

```shell
http://10.0.0.12
admin
```

### 3 镜像推送与下载

docker配置文件添加白名单

```shell
vim /etc/docker/daemon.json
"insecure-registries":["10.0.0.12"],  # 不要加端口,可以是IP或域名，中间逗号隔开可加多个
```

镜像打标签

```shell
docker pull 10.0.0.12/xxx/centos6.9_ssh:v2   # xxx是仓库中的创建的项目名
```

登录到仓库

```shell
docker login 10.0.0.12
```

推送到仓库

```shell
docker push 10.0.0.12/xxx/centos6.9_ssh:v2
```

从仓库下载镜像

```shell
docker pull 10.0.0.12/xxx/centos6.9_ssh:v2
```

### 4 将Harbor升级为https访问

配置文件修改，主要是添加证书路径

```shell
# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /your/certificate/path
  private_key: /your/private/key/path
```

再次执行安装脚本

```shell
./install.sh
```

