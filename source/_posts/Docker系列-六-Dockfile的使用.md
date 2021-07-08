---
title: Docker系列(六)-Dockfile的使用
date: 2021-06-30 08:43:52
categories:
- Docker
tags:
- Docker
- 学习笔记
---

## 一、Dokerfile简介

>Dockerfile 是一个用来自动构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
>
>建议存放在/opt/dockerfile中，如创建centosXX的镜像，则创建/opt/dockerfile/centosXX/Dockerfile

### 1 Dockerfile的简单使用

>创建一个开启sshd服务的centos6.9镜像

创建yum源文件，用于拷贝到centos6.9镜像中

```shell
cd /opt/dockfile/centos6.9_ssh
vim CentOS-Base.repo
...内容省略
```

创建Dockerfile文件/opt/dockfile/centos6.9_ssh/Dockerfile，内容如下：

```dockerfile
FROM centos:6.9
ADD CentOS-Base.repo /etc/yum.repos.d
RUN yum install openssh-server -y
RUN service sshd restart
RUN echo '123456' | passwd --stdin root
CMD ["/usr/sbin/sshd","-D"]
```

>PS：RUN的执行过程：创建临时容器，执行命令，提交成临时镜像，删除临时容器，重复此步骤。

构建镜像

```shell
docker build -t centos6.9_ssh:v2 /opt/dockfile/centos6.9_ssh
```

>PS：最后传入的是包含Dockerfile的文件夹，区分大小写，可以用"."代替 

验证镜像是否正常

```shell
docker run -d -p 1022:22 centos6.9_ssh:v2
```

>PS：最后不用接命令，将自动执行CMD指定的命令

### 2 小案例

>创建centos6.9 + ssh + nginx的Dockerfile

```shell
[root@docker01 centos6.9_ssh_nginx]# pwd
/opt/dockfile/centos6.9_ssh_nginx
[root@docker01 centos6.9_ssh_nginx]# ls
CentOS-Base.repo  Dockerfile  epel.repo  init.sh
```

```shell
# init.sh
#!/bin/bash
service sshd restart
nginx -g "daemon off;"
```

编写Dockerfile

```shell
FROM centos:6.9
ADD CentOS-Base.repo /etc/yum.repos.d
ADD epel.repo /etc/yum.repos.d
ADD init.sh /root
RUN yum install openssh-server nginx -y
RUN echo '123456' | passwd --stdin root
CMD ["/bin/bash","/root/init.sh"]
```

构建镜像

```she
docker build -t centos6.9_ssh_nginx:v3 .
```

测试使用

```shell
docker run -d -p 1022:22 -p 81:80 centos6.9_ssh_nginx:v3
```

## 二、Docker指令

| 命令       | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| FROM       | 基于那个镜像来构建                                           |
| MAINTAINER | 镜像的创建者                                                 |
| ENV        | 设置环境变量                                                 |
| ADD        | 添加宿主机文件到容器里，有需要解压的文件会自动解压           |
| COPY       | 添加宿主机文件到容器里                                       |
| WORKDIR    | 切换工作目录                                                 |
| EXPOSE     | 开放可用端口                                                 |
| CMD        | 容器启动后执行的命令，可被docker run指定的命令覆盖           |
| ENTRYPOINT | 容器启动后执行的命令，但不回被docker run指定的命令覆盖，如需覆盖，需要加--entrypoint参数 |
| VOLUME     | 创建挂载卷，将宿主机的目录挂载到容器里                       |

## 三、案例：Dockerfile构建可道云容器

>项目：
>
>​	可道云网盘kodexplorer
>
>环境：
>
>​	httpd+php或者nginx+php
>​	php所需模块：php5.5以上
>​	基础镜像：centos:7.9
>​	项目下载地址: http://static.kodcloud.com/update/download/kodexplorer4.37.zip
>​	项目官网：https://kodcloud.com/download/

### 1 手工部署一遍

>写Dockerfile前自己手动部署一遍，主要是nginx + php的搭建，参考博客
>
>https://cloud.tencent.com/developer/article/1015237

修改nginx.conf

```shell
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }

    	location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    	}
	
    	location / {
        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$args;
    	}
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

修改php-fpm.conf

```shell
;;;;;;;;;;;;;;;;;;;;;
; FPM Configuration ;
;;;;;;;;;;;;;;;;;;;;;

; All relative paths in this configuration file are relative to PHP's install
; prefix.

; Include one or more files. If glob(3) exists, it is used to include a bunch of
; files from a glob(3) pattern. This directive can be used everywhere in the
; file.
include=/etc/php-fpm.d/*.conf

;;;;;;;;;;;;;;;;;;
; Global Options ;
;;;;;;;;;;;;;;;;;;

[global]
; Pid file
; Default Value: none
pid = /run/php-fpm/php-fpm.pid

; Error log file
; Default Value: /var/log/php-fpm.log
error_log = /var/log/php-fpm/error.log

; Log level
; Possible Values: alert, error, warning, notice, debug
; Default Value: notice
;log_level = notice

; If this number of child processes exit with SIGSEGV or SIGBUS within the time
; interval set by emergency_restart_interval then FPM will restart. A value
; of '0' means 'Off'.
; Default Value: 0
;emergency_restart_threshold = 0

; Interval of time used by emergency_restart_interval to determine when 
; a graceful restart will be initiated.  This can be useful to work around
; accidental corruptions in an accelerator's shared memory.
; Available Units: s(econds), m(inutes), h(ours), or d(ays)
; Default Unit: seconds
; Default Value: 0
;emergency_restart_interval = 0

; Time limit for child processes to wait for a reaction on signals from master.
; Available units: s(econds), m(inutes), h(ours), or d(ays)
; Default Unit: seconds
; Default Value: 0
;process_control_timeout = 0

; Send FPM to background. Set to 'no' to keep FPM in foreground for debugging.
; Default Value: yes
daemonize = no

;;;;;;;;;;;;;;;;;;;;
; Pool Definitions ; 
;;;;;;;;;;;;;;;;;;;;

; See /etc/php-fpm.d/*.conf
```

修改www.conf

```shell
# 12行
listen = /var/run/php-fpm/php-fpm.sock
# 31-32行
listen.owner = nobody
listen.group = nobody
# 39-41行
user = nginx
; RPM: Keep a group allowed to write in log dir.
group = nginx
```

修改php.ini

```shell
cgi.fix_pathinfo 把它的值设置为 0
```

### 2 Dockerfile部署

文件存放

```shell
[root@docker01 centos7.9_kod]# ll
total 92
-rw-r--r--. 1 root root   661 Jul  2 11:21 Dockerfile
-rw-r--r--. 1 root root   171 Jul  1 14:28 init.sh
-rw-r--r--. 1 root root  2715 Jul  1 13:50 nginx.conf
-rw-r--r--. 1 root root  1691 Jul  1 13:50 php-fpm.conf
-rw-r--r--. 1 root root 64945 Jul  1 14:48 php.ini
-rw-r--r--. 1 root root 10029 Jul  1 13:50 www.conf
[root@docker01 centos7.9_kod]# pwd
/opt/dockfile/centos7.9_kod
```

编写Dockerfile

```dockerfile
FROM centos:7.9.2009
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo && \
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo && \
yum install openssh-server nginx net-tools php-cli php-fpm unzip php-gd php-mbstring  -y
ADD nginx.conf /etc/nginx/nginx.conf
ADD php-fpm.conf /etc/php-fpm.conf
ADD www.conf /etc/php-fpm.d/www.conf
ADD php.ini /etc
ADD init.sh /root/
EXPOSE 80 22
WORKDIR /usr/share/nginx/html
RUN curl -o kod.zip https://static.kodcloud.com/update/download/kodexplorer4.45.zip && \
unzip kod.zip && \
chmod -R 777 /usr/share/nginx/html/
CMD ["/bin/bash","/root/init.sh"]
```

构建镜像

```shell
docker build -t centos7.9_kod:v1 .
```

运行容器

```shell
docker run -d -p 80:80 -p 1022:22 -e "SSH_PWD=redhat123" --privileged centos7.9_kod:v1 /usr/sbin/init
```

进入容器，并运行初始化命令

```shell
docker exec -it 865 /bin/bash
# 容器启动服务，设置root密码
[root@88179198e672 html]#systemctl restart sshd php-fpm nginx
[root@88179198e672 html]#echo "redhat123" | passwd --stdin root
```

测试访问：

```shell
网页访问：http://10.0.0.11可进去可道云界面
ssh 10.0.0.11 -p1022 可以登录镜像
```





