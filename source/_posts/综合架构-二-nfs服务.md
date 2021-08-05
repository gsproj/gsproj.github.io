---
title: 综合架构(二)-nfs服务
date: 2021-07-30 16:16:32
categories:
- NFS
- 综合架构
tags:
- NFS
- 学习笔记
---

## 一、NFS简介

>NFS是Network File System的缩写及网络文件系统。NFS主要功能是通过局域网络让不同的主机系统之间可以共享文件或目录。
>NFS系统和Windows网络共享、网络驱动器类似, 只不过windows用于局域网, NFS用于企业集群架构中
>如果是大型网站, 会用到更复杂的分布式文件系统FastDFS（音频、小说、视频）,glusterfs（iso镜像）,HDFS
>NFS（图片、）解决共享前端web共享

### 1.1 NFS有什么用？

解决前端web静态资源的共享
解决前端web静态资源一致性
解决前端web磁盘空间的浪费

### 1.2 NFS的文件操作方式

1.当用户执行mkdir命令, 该命令会调用shell解释器翻译给内核。
2.内核解析完成后会驱动对应的硬件设备，完成相应的操作。

### 1.3 NFS的实现原理

1.用户进程访问NFS客户端，使用不同的函数对数据进行处理
2.NFS客户端通过TCP/IP的方式传递给NFS服务端。
3.NFS服务端接收到请求后，会先调用portmap进程进行端口映射。
4.nfsd进程用于判断NFS客户端是否拥有权限连接NFS服务端。
5.Rpc.mount进程判断客户端是否有对应的权限进行验证。
6.idmap进程实现用户映射和压缩
7.最后NFS服务端会将对应请求的函数转换为本地能识别的命令，传递至内核，由内核驱动硬件。
注意: rpc是一个远程过程调用，那么使用nfs必须有rpc服务

### 1.4 NFS的优缺点

优点

```shell
1.NFS文件系统简单易用、方便部署、数据可靠、服务稳定、满足中小企业需求。
2.NFS文件系统内存放的数据都在文件系统之上，所有数据都是能看得见。
```

缺点

```shell
1.存在单点故障, 如果构建高可用维护麻烦
web->nfs(sersync)->backup
2.NFS数据明文, 并不对数据做任何校验。
3.客户端挂载NFS服务没有密码验证, 安全性一般(内网使用)
```

>3.NFS应用建议
>1.生产场景应将静态数据尽可能往前端推, 减少后端存储压力
>2.必须将存储里的静态资源通过CDN缓存jpg\png\mp4\avi\css\js
>3.如果没有缓存或架构本身历史遗留问题太大, 在多存储也无用

## 二、NFS部署

### 2.1 服务端

安装

```shell
[root@nfs ~]# yum install nfs-utils -y
```

配置

```shell
[root@nfs ~]# cat /etc/exports
/data 172.16.1.0/24(rw,sync,all_squash)
```

授权

```shell
[root@nfs ~]# mkdir /data
[root@nfs ~]# chown -R nfsnobody.nfsnobody /data/
```

启动服务

```shell
[root@nfs ~]# systemctl start nfs-server
[root@nfs ~]# systemctl enable nfs-server
```

检查

```shell
[root@nfs ~]# cat /var/lib/nfs/etab 
/data	172.16.1.0/24(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,rw,secure,root_squash,all_squash)
```

### 2.2 客户端

安装

```shell
[root@nfs ~]# yum install nfs-utils -y
```

检查nfs是否有共享内容

```shell
[root@web01 ~]# showmount -e 172.16.1.31
Export list for 172.16.1.31:	
/data 172.16.1.0/24
```

挂载nfs目录（临时）

```shell
[root@web01 ~]# mount -t nfs 172.16.1.31:/data /opt
[root@web01 ~]# df -h
Filesystem         Size  Used Avail Use% Mounted on
172.16.1.31:/data   99G  1.8G   98G   2% /opt
```

挂载nfs目录（永久）

```shell
[root@web01 ~]# vim /etc/fstab 
172.16.1.31:/data  /opt  nfs  defaults  0  0
[root@web01 ~]# mount -a   #验证fstab开机启动是否填写错误。
```

### 2.3 NFS配置参数说明

```shell
ro				只读权限
root_squash		当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户(不常用)
no_root_squash	当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员(不常用)
no_all_squash	无论NFS客户端使用什么账户访问，都不进行压缩
async			优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据

rw*				读写权限
sync*			同时将数据写入到内存与硬盘中，保证不丢失数据
all_squash*		无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户(常用)
anonuid*		配置all_squash使用,指定NFS的用户UID,必须存在系统
anongid*		配置all_squash使用,指定NFS的用户UID,必须存在系统
```

### 2.4 验证ro权限

```shell
[root@nfs ~]# cat /etc/exports
/data 172.16.1.0/24(ro,sync,all_squash)
[root@nfs ~]# systemctl restart nfs-server

[root@web01 ~]# touch /opt/ttt
touch: cannot touch ‘/opt/ttt’: Read-only file system		#通常这样的错误都是设定的ro权限导致
```

### 2.5 验证all_squash、anonuid、anongid权限

服务端

```shell
[root@nfs ~]# cat /etc/exports
/data 172.16.1.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
创建用户
[root@nfs ~]# groupadd -g 666 www
[root@nfs ~]# useradd -u666 -g666 www
[root@nfs ~]# id www
uid=666(www) gid=666(www) groups=666(www)
授权
[root@nfs ~]# chown -R www.www /data/
重启
[root@nfs ~]# systemctl restart nfs-server
```

客户端重新挂载验证

```shell
[root@web01 opt]# touch file
[root@web01 opt]# touch test
[root@web01 opt]# ll
total 4
-rw-r--r-- 1 666 666 0 Jan  7 11:29 file
-rw-r--r-- 1 666 666 0 Jan  7 11:29 test
```

>PS:由于客户端没有创建id为666的www用户，因此看到的属组和属主都是666

客户端如果觉得666不好看，建议在客户端上创建同名的用户以及uid

```shell
[root@web01 ~]# groupadd -g 666 www
[root@web01 ~]# useradd -u 666 -g 666 www
[root@web01 ~]# id www
uid=666(www) gid=666(www) groups=666(www)
[root@web01 ~]# ll /opt/
total 4
-rw-r--r-- 1 www www 0 Jan  7 11:29 file
-rw-r--r-- 1 www www 0 Jan  7 11:29 test
```

