---
title: 综合架构(一)整体架构规划与Rsync
date: 2021-07-29 14:20:37
categories:
- Rsync
- 综合架构
tags:
- Rsync
- 学习笔记
---


## 一 综合架构规划

>​	一个项目涵盖了一套架构，一套架构又涵盖了不同的角色（高可用、负载均衡、web集群）
>五层架构模型--> 负载均衡 web服务 存储服务 缓存服务 数据库服务（通过tcp连接）

{% asset_img 综合架构图.png 综合架构图 %}

![综合架构图](D:\学习视频\老男孩上海3期（主看）\老男孩教育-上海linux脱产3期-02综合架构课程视频\day24 老男孩教育3期-综合架构\综合架构图.png)

### 1 架构访问流程 

#### 1.1 用户视角

```shell
1.用户通过浏览器输入oldboyedu.com->回车
2.浏览器会发生一次跳转，分析URL->然后进行DNS解析->获取真实的公网IP地址
3.用户通过tcp的三次握手发起连接->真实的公网IP
4.连接会通过公网->路由器->交换机->抵达前端的硬件防火墙
5.防火墙根据自身访问规则，进行匹配->如果恶意的连接则拒绝->如果是正常的连接则放行
6.防火墙会将连接转发给负载均衡器->查看用户请求的内容->根据内容进行任务下发->下发给web服务器
7.web服务接收请求后会根据请求进行判断
如果是请求图片或者附件->查找存储服务器存储的静态资源
如果请求的网站上的内容->缓存服务器->如果缓存服务器没有->数据库
数据库查询完数据之后会返回数据给web服务器->同时也会返回一份给缓存服务器
8.数据库返回内容->web服务器->负载均衡->用户
```

#### 1.2 运维视角

```shell
1.用户通过公网连接（隧道）VPN服务器，这样方便管理内部主机，
2.自动化配置管理，节省人力成本，便于后期维护。统一环境，标准化
3.自动化监控服务，监控系统的运行状态，事前预警，事后追溯。
```

### 2 架构环境规划

#### 2.1 IP分配

```shell
 wanip         lanip       hostname
10.0.0.5     172.16.1.5     lb01
10.0.0.6     172.16.1.6     lb02
10.0.0.7     172.16.1.7     web01
10.0.0.8     172.16.1.8     web02
10.0.0.9     172.16.1.9     web03
10.0.0.31    172.16.1.31    nfs
10.0.0.41    172.16.1.41    backup
10.0.0.51    172.16.1.51    db01
10.0.0.61    172.16.1.61    m01
10.0.0.71    172.16.1.71    zabbix
```

#### 2.2 基础环境准备

#### 2.2.1 虚拟机系统和网卡准备

>安装全新Centos7系统，配置网卡为eth0及eth1命名模式
>    1.第一块网卡为NAT模式[公网环境]，配置的网段为10.0.0.0网段
>    2.第二块网卡为LAN模式[私网环境]，配置的网段为172.16.1.0网段
>    3.优化安装好的Centos7虚拟机，安装常用软件、关闭防火墙等等

优化步骤

```shell
# 1.配置yum仓库
rm -f /etc/yum.repos.d/*
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# 2.安装基础软件包
yum install net-tools vim tree htop iftop \
iotop lrzsz sl wget unzip telnet nmap nc psmisc \
dos2unix bash-completion bash-completion-extra sysstat \
rsync nfs-utils httpd-tools -y

# 3.关闭防火墙firewalld
systemctl disable firewalld
systemctl stop firewalld

# 4.关闭selinux
sed -i '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config

# 5.调整单个进程最大能打开文件的数量
echo '* - nofile 65535' >> /etc/security/limits.conf
```

基于优化后的虚拟机进行克隆

```shell
1.连接克隆（需要依赖于母体）
2.完整克隆（完完全全的复制一份，占用磁盘空间）
```

对新克隆的主机进行如下操作：

```shell
# 1.修改主机名  
hostnamectl set-hostname backup

# 2.修改IP地址  
sed -i 's#200#41#g' /etc/sysconfig/network-scripts/ifcfg-eth[01]

# 3.重启服务器
```

## 二、Rsync与数据备份

### 2.1  备份概念

为什么要做备份？

```shell
数据非常的重要
保证数据不丢失
便于快速的恢复
```

备份方式

```shell
完全备份，每次都进行全部备份 (效率低下, 占用空间)
增量备份，仅备份客户端与服务端差异的部分 (提高备份效率,节省空间, 适合异地备份 )
```

用什么工具做备份？

```shell
scp  	网络之间的拷贝，全量拷贝的方式  （ssh协议）
rsync	远程同步（增量）
```

### 2.2 rsync的基本概念

>rsync是一款开源的备份工具，
>	可以在不同主机之间进行同步（windows和Linux之间   Mac和Linux   Linux和Linux）
>	可实现全量备份与增量备份
>	因此非常适合用于架构集中式备份或异地备份等应用

rsync数据的同步模式

```shell
推送： 本地将数据上传至备份服务器上	 （上传）
拉取： 备份服务器获取本地服务器的数据  （下载）
```

rsync的数据传输方式

```shell
本地传输（类似于使用cp命令）
远程传输（通过网络传输  a-->b）
守护进程（运行一个服务一直在后台）
```

rsync选项详解

```shell
rsync参数:
-a           #归档模式传输, 等于-tropgDl *
-v           #详细模式输出, 打印速率, 文件数量等 *
-z           #传输时进行压缩以提高效率 *
-r           #递归传输目录及子目录，即目录下得所有目录都同样传输。
-t           #保持文件时间信息
-o           #保持文件属主信息
-p           #保持文件权限
-g           #保持文件属组信息
-l           #保留软连接
-P           #显示同步的过程及传输时的进度等信息
-D           #保持设备文件信息
-L           #保留软连接指向的目标文件
-e           #使用的信道协议,指定替代rsh的shell程序  ssh
--exclude=PATTERN   #指定排除不需要传输的文件模式
--exclude-from=file #文件名所在的目录文件
--bwlimit=100       #限速传输 *
--partial           #断点续传
--delete            #让目标目录和源目录数据保持一致 *
```

### 2.3 本地传输

将/boot文件夹拷贝到/tmp中

```shell
rsync -avz /boot /tmp/
```

将boot文件夹中的内容拷贝到/tmp中

```shell
rsync -avz /boot/ /tmp/
```

### 2.3 远程传输

虚拟机准备

| 主机名 | IP地址                         |
| ------ | ------------------------------ |
| nfs    | 172.16.1.31/24 和 10.0.0.31/24 |
| backup | 172.16.1.41/24 和 10.0.0.41/24 |

backup从nfs拉取/boo目录到本地/tmp文件夹

```she
rsync -avz root@10.0.0.31:/boot /tmp
```

backup上传/root/test.txt到nfs主机的/tmp文件夹中

```shell
rsync /root/test.txt root@10.0.0.31:/tmp
```

### 2.4 守护进程传输

>为什么需要使用rsync守护进程传输？
>
>	Rsync借助SSH协议同步数据存在的缺陷（临时发送数据）
>		1.使用系统用户（不安全）
>		2.使用普通用户（会导致权限不足情况）

backup充当rsync服务端，nfs充当客户端，配置步骤：

#### 2.4.1 服务端配置

获取配置文件路径

```shell
[root@backup ~]# rpm -qc rsync
	/etc/rsyncd.conf			# 主配置文件
	/etc/sysconfig/rsyncd		# 选项
```

编辑配置文件

```shell
vim /etc/rsyncd.conf
---------------------
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
ignore errors
read only = false
list = false
auth users = rsync_backup
secrets file = /etc/rsync.passwd
log file = /var/log/rsyncd.log
#####################################
[backup]
comment = welcome to oldboyedu backup!
path = /backup
```

配置详细解析

```shell
uid = rsync                      # 运行进程的用户
gid = rsync                      # 运行进程的用户组
port = 873                       # 监听端口
fake super = yes                 # 无需让rsync以root身份运行，允许接收文件的完整属性
use chroot = no                  # 禁锢推送的数据至某个目录, 不允许跳出该目录
max connections = 200            # 最大连接数
timeout = 600                    # 超时时间
ignore errors                    # 忽略错误信息
read only = false                # 对备份数据可读写
list = false                     # 不允许查看模块信息
auth users = rsync_backup        # 定义虚拟用户，作为连接认证用户
secrets file = /etc/rsync.passwd # 定义rsync服务用户连接认证密码文件路径
log file = /var/log/rsyncd.log   # 日志文件
#####################################
[backup]                # 定义模块信息
comment = commit        # 模块注释信息
path = /backup          # 定义接收备份数据目录
```

创建rsync进程启动时需要使用的用户

```shell
[root@backup ~]# useradd rsync -M -s /sbin/nologin 
[root@backup ~]# id rsync
uid=1000(rsync) gid=1000(rsync) groups=1000(rsync)
```

创建密码文件，在密码文件中写入对应的虚拟用户以及虚拟用户的密码

```shell
/etc/rsync.passwd---》rsync虚拟用户以及rsync虚拟用户的密码
[root@backup ~]# echo "rsync_backup:123456" > /etc/rsync.passwd
[root@backup ~]# chmod 600 /etc/rsync.passwd 
```

创建存储备份数据的目录，并进行授权

```shell
[root@backup ~]# mkdir /backup
[root@backup ~]# chown -R rsync.rsync /backup/
```

启动rsync服务并加入开机自启动

```shell
[root@backup ~]# systemctl start rsyncd.service 
[root@backup ~]# systemctl enable rsyncd
```

检查rsync的873端口是否存在

```shell
[root@backup ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp6       0      0 :::873                  :::*                    LISTEN      1269/rsync
```

#### 2.4.2 客户端测试

推送/etc文件夹到服务端/backup

```shell
rsync -avz /etc/ rsync_backup@172.16.1.41::backup
需要输入密码
```

从服务端/backup拉取文件到/tmp

```shell
rsync -avz rsync_backup@172.16.1.41::backup /tmp
需要输入密码
```

### 2.5 rsync补充

#### 2.5.1 无差异同步(慎用)

```shell
#推送方式实现无差异，以客户端为准，客户端有什么服务端就有什么
[root@nfs ~]# rsync -avz --delete /root rsync_backup@172.16.1.41::backup		

#拉取方式实现无差异，以服务端为准，服务端有什么客户端就有什么
[root@nfs ~]# rsync -avz --delete rsync_backup@172.16.1.41::backup /opt/
```

#### 2.5.2 传输限速

```shell
# 生成大文件
[root@nfs ~]# dd if=/dev/zero of=./size.disk bs=1M count=500  

# 限制传输的速率为1MB 
[root@nfs ~]# rsync -avzP --bwlimit=1 ./size.disk rsync_backup@172.16.1.41::backup
Password: 
sending incremental file list
size.disk
    118,358,016  22%    1.01MB/s    0:06:33
```

#### 2.5.3 取消每次传输需要输密码

在客户端配置

```shell
方式一：
[root@nfs ~]# echo "123456" > /etc/rsync.pass
[root@nfs ~]# chmod 600 /etc/rsync.pass #上该文件找123456
[root@nfs ~]# rsync -avzP --bwlimit=1 ./size.disk rsync_backup@172.16.1.41::backup --password-file=/etc/rsync.pass

方式二：写Shell脚本
[root@nfs ~]# export RSYNC_PASSWORD=123456
[root@nfs ~]# rsync -avzP ./size.disk rsync_backup@172.16.1.41::backup
```

## 三、Rsync备份案例

>客户端需求
>1.客户端提前准备存放的备份的目录，目录规则如下:/backup/nfs_172.16.1.31_2018-09-02 
>2.客户端在本地打包备份(系统配置文件、应用配置等)拷贝至/backup/nfs_172.16.1.31_2018-09-02
>3.客户端最后将备份的数据进行推送至备份服务器
>4.客户端每天凌晨1点定时执行该脚本
>5.客户端本地保留最近7天的数据, 避免浪费磁盘空间
>
>服务端需求
>1.服务端部署rsync，用于接收客户端推送过来的备份数据
>2.服务端需要每天校验客户端推送过来的数据是否完整
>3.服务端需要每天校验的结果通知给管理员
>4.服务端仅保留6个月的备份数据,其余的全部删除
>
>注意：所有服务器的备份目录必须都为/backup
>
>1.客户端将需要备份的文件放入指定的目录中   /backup/nfs_172.16.1.31_2018-09-02
>2.客户端每天凌晨1点使用rsync命令推送一次nfs_172.16.1.31_2018-09-0
>3.客户端保留最近7天的数据即可

### 3.1 需求分析

>1.我要备份什么？
>	/etc/fstab /var/spool/cron/USERNAME   /server/scripts
>
>2.我要怎么备份？
>	/backup/主机名_ip地址_时间  命名的目录中
>
>3.我要备份到哪？
>	rsync备份服务器   172.16.1.41

#### 3.1.1 服务端配置

配置邮件服务

```shell
[root@backup ~]# yum install mailx -y
[root@backup ~]# vim /etc/mail.rc
set from=12345@qq.com
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=12345@qq.com
set smtp-auth-password=xxxxxx # 授权码
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/
```

脚本编写

```shell
[root@backup ~]# mkdir /server/scripts -p
[root@backup ~]# cat /server/scripts/check_client_data.sh
#!/bin/bash
#1.定义变量
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
SRC=/backup
DATE=$(date +%F)

#1.使用md5进行校验，并保存校验的结果
md5sum -c $SRC/*_$DATE/flag_$DATE > $SRC/result_$DATE

#2.将保存的结果文件发送给管理员
mail -s "Rsync Backup $DATE" 572891887@qq.com <$SRC/result_$DATE

#3.保留最近180天的数据
find $SRC/ -type d -mtime +180|xargs rm -rf 
```

#### 3.1.2 客户端配置

创建目录

```shell
mkdir /backup
mkdir -p /server/scripts/
```

编写备份脚本

```shell
[root@nfs ~]# cat /server/scripts/client_push_data.sh
#!/bin/bash
#1.定义变量
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
SRC=/backup
HOST=$(hostname)
ADDR=$(ifconfig eth1|awk 'NR==2 {print $2}')
DATE=$(date +%F)
DEST=${HOST}_${ADDR}_${DATE}

#2.创建目录
[ -d $SRC/$DEST ] || mkdir -p $SRC/$DEST

#3.备份文件
cd / && \
[ -f $SRC/$DEST/sys.tar.gz ] || tar czf $SRC/$DEST/sys.tar.gz etc/fstab etc/passwd && \
[ -f $SRC/$DEST/other.tar.gz ] || tar czf $SRC/$DEST/other.tar.gz var/spool/cron/ server/scripts && \

#4.使用md5打标记
[ -f $SRC/$DEST/flag_$DATE ] || md5sum $SRC/$DEST/*.tar.gz  > $SRC/$DEST/flag_$DATE 

#4.本地推送到备份服务器
export RSYNC_PASSWORD=123456
rsync -avz $SRC/$DEST rsync_backup@172.16.1.41::backup

#5.保留本地最近7天的数据
find $SRC/ -type d -mtime +7|xargs rm -rf 
```

#### 3.1.3 整体测试:设置定时任务

客户端每两分钟备份推送一次

```shell
[root@nfs backup]# crontab -e
*/2 * * * * sh /server/scripts/client_push_data.sh
```

服务端每3分钟校验一次，并发送确认邮件

```shell
[root@backup scripts]# crontab -e
*/3 * * * * sh /server/scripts/check_client_data.sh
```

#### 3.1.5 增加客户端数量

```shell
多创建几台客户端服务器，测试
```

