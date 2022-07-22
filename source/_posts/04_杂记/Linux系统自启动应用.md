---
title: Linux系统自启动应用
date: 2022-07-22 11:02:52
categories:
- 杂记
tags:
---



>“如何实现开机启动自定义的应用”

## 1、创建rc-local服务文件

创建文件

```python
sudo vim /etc/systemd/system/rc-local.service
```

内容：

```shell
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99
[Install]
WantedBy=multi-user.target
```

## 2、激活rc-local服务

```shell
sudo systemctl enable rc-local.service	
```

## 3、添加服务文件  

```shell
sudo vim /etc/rc.local
```

内容

```shell
#!/bin/sh -e
# #
rc.local
# #
This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
# #
In order to enable or disable this script just change the execution
# bits.
# #
By default this script does nothing.

# 下面是要开机启动的命令
# 启动nginx
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
# 启动redis
/home/sofware/redis/redis-4.8.0/src/redis-server /home/sofware/redis/redis-4.8.0/redis.conf
# 启动nacos
/bin/bash -f /home/Software/nacos/startup.sh -m standalone
exit 0
```

给脚本文件添加执行权限  

```shell
#给予脚本执行权限
sudo chmod +x /etc/rc.local
```

## 4、启动rc-local服务  

```shell
systemc start rc-local.service
```

## 5、查看服务启动情况  

```shell
systemctl status rc-local
```

正常情况下是Active状态显示active(exited)  



