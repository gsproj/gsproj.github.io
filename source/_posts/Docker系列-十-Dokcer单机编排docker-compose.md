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

docker01主机创建docker-compose文件

```shell
vim /opt/docker-compose/wordpress/docker-compose.yml
```

```yaml
version: '3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - web_data:/var/www/html
    ports:
     - "80"  # 随机端口映射到内网80端口
    restart: always
    environment:
     WORDPRESS_DB_HOST: db:3306
     WORDPRESS_DB_USER: wordpress
     WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
  web_data:
```

构建Docker容器(三个wordpress，一个mysql）

```shell
docker-compose up --scale wordpress=3 -d
-d 后台运行
--scale 生成的实例数
```

```she
netstaus -lntup可以看到3个连续的端口号
tcp        0      0 0.0.0.0:49156           0.0.0.0:*               LISTEN      2110/docker-proxy
tcp        0      0 0.0.0.0:49157           0.0.0.0:*               LISTEN      2126/docker-proxy
tcp        0      0 0.0.0.0:49158           0.0.0.0:*               LISTEN      2143/docker-proxy
```

docker02主机安装nginx，用于负载均衡

```she
curl -o epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
yum install nginx
```

配置nginx

```she
cd /etc/nginx/
mv nginx.conf nginx.conf.bak
grep -Ev '^$|#' nginx.conf.default  > nginx.conf
vim nginx.conf
```

```shell
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    upstream wordpress {
        server 10.0.0.11:49156;
        server 10.0.0.11:49157;
        server 10.0.0.11:49158;
    }

    server {
        listen       80;
        server_name  localhost;
        location / {
            proxy_pass  http://wordpress;
            proxy_set_header Host $host;  # 不加上网页没有Host信息，显示不全
        }
    }
}
```

重启nginx服务

```shell
nginx -t
systemctl restart nginx
```

测试访问

```she
浏览器访问:10.0.0.12,可以进入wordpress
```

查看是否负载均衡

```shell
# 创建一个查看信息的页面
cd /var/lib/docker/volumes/wordpress_web_data/_data/
vim info.php
```

```php
<?php phpinfo(); ?>
```

浏览器访问：10.0.0.12/info.php，查看其中IP信息

## 二、案例：构建zabbix