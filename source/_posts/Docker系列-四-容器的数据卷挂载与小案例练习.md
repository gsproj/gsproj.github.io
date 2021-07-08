---
title: Docker系列(四)-容器的数据卷挂载与小案例练习
date: 2021-06-29 10:31:05
categories:
- Docker
tags:
- Docker
- 学习笔记
---

## 一、数据卷挂载

### 1 临时挂载

```shell
# 将/opt/xiaoniao目录挂载到容器的html目录
docker run -d -p 80:80 -v /opt/xiaoniao:/usr/share/nginx/html nginx:latest
```

### 2 使用卷挂载

>容器被删除，创建的卷可以保留，可以再次挂载到新建的容器中

创建名为myvol的容器卷并挂载到容器html目录

```shell
docker run -d -p 80:80 -v myvol:/usr/share/nginx/html nginx:latest
```

查看当前有哪些容器卷

```shell
docker volume ls
```

查看名为myvol的卷的信息

```shell
docker volume inspect myvol
```

删除容器并删除卷（无效）

```shell
# PS：删除容器并删除卷，无法将卷删除
docker rm -f -v [容器ID]
    -v --volume
```

## 二、小案例：多端口多站点

>80端口访问nginx首页
>
>81端口访问水果忍者

获取水果忍者HTML5小游戏源码

```shell
wget https://7npmedia.w3cschool.cn/1-FruitNinja.7z
```

创建81端口nginx配置文件

```shell
vim /opt/fruitninjia.conf
server {
    listen       81;
    listen  [::]:81;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /data;
        index  index.html index.htm;
    }
}
```

将游戏源码文件解压至/opt/fruitninjia，并运行nginx容器

```shell
# 将/opt/fruitninjia挂载到容器/data中
docker run -d -p 80:80 -p 81:81 -v /opt/fruitninjia:/data -v /opt/fruitninjia.conf:/etc/nginx/conf.d/fruitninjia.conf nginx
```

网页访问

```shell
https://10.0.0.11:80
```

