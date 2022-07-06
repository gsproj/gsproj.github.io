---
title: 运维之基础命令--yum仓库
date: 2022-07-06 11:19:52
categories:
- 运维
- （一）基础命令
tags:
---

# yum仓库

1、克隆两台主机

```bash
yum仓库主机
	外网地址：192.168.15.30
	内网地址：172.16.1.30

yum测试主机
	外网地址：192.168.15.31
	内网地址：172.16.1.31

修改的命令：
	vim /etc/sysconfig/network-scripts/ifcfg-eth0
	vim /etc/sysconfig/network-scripts/ifcfg-eth1
	
	重启网卡：systemctl restart network
	
	修改主机名：
		yum仓库主机：hostnamectl set-hostname warehouse
		yum测试主机：hostnamectl set-hostname yum-test
	
```

2、配置yum仓库

```bash
1、需要有一个软件包目录，存放软件包的
[root@warehouse ~]# mkdir /backup
	
	1.1、缓存yum安装下载的软件包
	[root@warehouse 7]# vim /etc/yum.conf
	[main]
    cachedir=/var/cache/yum/$basearch/$releasever
    keepcache=1
    [root@warehouse ~]# yum install mariadb -y
	
	1.2、将缓存的软件包复制到yum仓库目录
	[root@warehouse ~]# cd /var/cache/yum/x86_64/7/base/packages/
	[root@warehouse packages]# cp -rp ./* /backup/
    [root@warehouse packages]# cd /backup/
    [root@warehouse backup]# ll
    total 8964
    -rw-r--r-- 1 root root 9175948 Oct 15 02:55 mariadb-5.5.68-1.el7.x86_64.rpm
	
2、建立软件包依赖关系
	[root@warehouse backup]# yum install createrepo -y
	[root@warehouse backup]# createrepo /backup/
    Spawning worker 0 with 1 pkgs
    Spawning worker 1 with 0 pkgs
    Workers Finished
    Saving Primary metadata
    Saving file lists metadata
    Saving other metadata
    Generating sqlite DBs
    Sqlite DBs complete
    [root@warehouse backup]# ll
    total 8968
    -rw-r--r-- 1 root root 9175948 Oct 15 02:55 mariadb-5.5.68-1.el7.x86_64.rpm
    drwxr-xr-x 2 root root    4096 Mar 18 08:54 repodata
    [root@warehouse backup]# ll repodata/
    total 28
    -rw-r--r-- 1 root root 1970 Mar 18 08:54 5d54624b2aa7a1fe974ff5553a9a78289683189c6a7b60c8c1421ecf011d270f-other.sqlite.bz2
    -rw-r--r-- 1 root root 3449 Mar 18 08:54 5e68ee34f7e8f1a11a039f55c990bcc044f16bceb6af7f4486b55d518ad91346-primary.sqlite.bz2
    -rw-r--r-- 1 root root  539 Mar 18 08:54 6e94b0fa1d98955542fb8238348ea171be7a130ba97573eb3ac756e1aadcec50-filelists.xml.gz
    -rw-r--r-- 1 root root 1291 Mar 18 08:54 a9afb0b2457f41f5c2d64744a07d5d12599e6c2588c870ae73568cf345a0abca-other.xml.gz
    -rw-r--r-- 1 root root 1333 Mar 18 08:54 cf41627875b17ef4bea5f0ea566ac63a5c83adc606d3ca730a74944ceb969028-primary.xml.gz
    -rw-r--r-- 1 root root 1158 Mar 18 08:54 f34cbfd9e5608e8402041e98100f853e913f3df04d6946874c73ba31a78969ba-filelists.sqlite.bz2
    -rw-r--r-- 1 root root 2969 Mar 18 08:54 repomd.xml
    [root@warehouse backup]# 
```

3、测试连接（本地版本）

```bash
1、备份系统所有的yum源
[root@warehouse yum.repos.d]# cd /etc/yum.repos.d/
[root@warehouse yum.repos.d]# mkdir backup1
[root@warehouse yum.repos.d]# mv *.repo backup1
[root@warehouse yum.repos.d]# vim local.repo
[local]
name="This is xxx"
baseurl=file:///backup
gpgcheck=0
enabled=1

# 清理yum缓存
[root@warehouse ~]# yum clean all
[root@warehouse ~]# rm -rf /var/cache/yum/x86_64/7/*

# 生成新的yum缓存
[root@warehouse ~]# yum makecache

# 测试连接
[root@warehouse yum.repos.d]# rpm -e mariadb
[root@warehouse yum.repos.d]# rpm -ql mariadb
package mariadb is not installed
[root@warehouse yum.repos.d]# yum install mariadb
```



4、测试连接（远程版本）

```bash
#########################  yum仓库机器上执行  ###################################
1、备份系统所有的yum源
[root@warehouse yum.repos.d]# cd /etc/yum.repos.d/
[root@warehouse yum.repos.d]# mkdir local
[root@warehouse yum.repos.d]# mv *.repo local

2、安装远程访问软件
[root@warehouse yum.repos.d]# cd /etc/yum.repos.d/
[root@warehouse yum.repos.d]# mv backup1/*  .
[root@warehouse yum.repos.d]# yum install vsftpd -y
[root@warehouse yum.repos.d]# systemctl enable --now vsftpd 
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.

3、配置系统yum仓库
[root@warehouse yum.repos.d]# cd /var/ftp
[root@warehouse ftp]# mkdir yum_warehouse

4、将软件包复制到yum仓库目录
[root@warehouse ftp]# cp -rp /backup/* /var/ftp/yum_warehouse/
[root@warehouse ftp]# ll yum_warehouse/
total 8968
-rw-r--r-- 1 root root 9175948 Oct 15 02:55 mariadb-5.5.68-1.el7.x86_64.rpm
drwxr-xr-x 2 root root    4096 Mar 18 08:54 repodata

5、建立yum软件包依赖关系
[root@warehouse ftp]# createrepo /var/ftp/yum_warehouse/


#########################  yum测试机器上执行  ###################################
1、本机测试网络连接
    1、备份系统所有的yum源
    [root@warehouse ftp]# cd /etc/yum.repos.d/
    [root@warehouse yum.repos.d]# mv *.repo backup1
    [root@warehouse yum.repos.d]# vim local.repo
    [root@warehouse yum.repos.d]# cat local.repo 
    [local-ftp]
    name="This is ftp server"
    baseurl=ftp://172.16.1.30/yum_warehouse
    gpgcheck=0
    enabled=1

    # 清理yum缓存
    [root@warehouse ~]# yum clean all
    [root@warehouse ~]# rm -rf /var/cache/yum/x86_64/7/*

    # 生成新的yum缓存
    [root@warehouse ~]# yum makecache

    # 测试连接
    [root@warehouse yum.repos.d]# rpm -e mariadb
    [root@warehouse yum.repos.d]# rpm -ql mariadb
    package mariadb is not installed
    [root@warehouse yum.repos.d]# yum install mariadb

2、yum测试机器执行
	1、备份yum仓库内容
	[root@yum-test ~]# cd /etc/yum.repos.d/
    [root@yum-test yum.repos.d]# mkdir backup1
    [root@yum-test yum.repos.d]# mv *.repo backup1
    [root@yum-test yum.repos.d]# ll
    total 0
    drwxr-xr-x. 2 root root 187 Mar  4 09:55 backup
    drwxr-xr-x  2 root root 220 Mar 18 09:20 backup1
    [root@yum-test yum.repos.d]# 
    
    2、编写本地yum源配置文件
    [root@yum-test yum.repos.d]# vim local.repo
    [root@yum-test yum.repos.d]# cat local.repo 
    [loacl-ftp-30]
    name="This is 30 ftp server"
    baseurl=ftp://172.16.1.30/yum_warehouse
    gpgcheck=0
    enabled=1
    
    3、清理yum缓存
    [root@yum-test yum.repos.d]# yum clean all
    [root@yum-test yum.repos.d]# rm -rf /var/cache/yum/x86_64/7/*
    
    4、生成yum缓存
    [root@yum-test yum.repos.d]# yum makecache
    
    5、测试安装
    [root@yum-test yum.repos.d]# yum install mariadb
```



4、同步远程yum仓库内容到本机

```bash
1、安装华为云镜像仓库
[root@warehouse yum.repos.d]# rm -rf /etc/yum.repos.d/*
[root@warehouse yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo

2、生成yum缓存
[root@warehouse yum.repos.d]# yum clean all
[root@warehouse yum.repos.d]# rm -rf /var/cache/yum/x86_64/7/*
[root@warehouse yum.repos.d]# yum makecache

3、同步华为云镜像站软件包到本地yum仓库
[root@warehouse ftp]# yum install yum-utils -y
[root@warehouse ftp]# reposync -r (仓库名称：yum repolist)

4、建立依赖关系
[root@warehouse ftp]# createrepo base

5、测试
    [loacl-ftp-30]
    name="This is 30 ftp server"
    baseurl=ftp://172.16.1.30/base
    gpgcheck=0
    enabled=1
```

![image-20210318112338781](.\assets\image-20210318112338781.png)





