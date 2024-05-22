---
title: day58-自动化架构-shell自动化编程（六）-完结
date: 2024-5-22 15:23:52
categories:
- 运维
- （二）综合架构
tags: 
---

# 自动化架构-shell自动化编程（六）-完结

今日内容：

​	Shell编程案例

# 一、Shell编程基础

## 1.0 通用函数库

后面都用得上

```shell
[root@mn01[ /server/scripts/devops-shell/shell-108]#cat 01_diy_func.sh 
#!/bin/bash
##############################################################
# File Name:01_diy_func.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc:
##############################################################


redecho() {
	#颜色开头部分
	echo -ne "\e[5;31m"
	#取出要加上颜色的内容
	echo -n "$@"
	#颜色的结束部分
	echo -e "\e[0m"
	#echo -e "\e[5;31m $@ \e[0m"
}


greenecho() {
	echo -ne "\e[1;32m"
	echo -n "$@"
	echo -e "\e[0m"
}

blueecho() {
	echo -ne "\e[1;34m"
	echo -n "$@"
	echo -e "\e[0m"
}

check_return_value() {
	if [ $? -eq 0 ]
	then
		greenecho $1 命令执行成功
	else
		redecho $1 命令执行失败
	fi
}

check_user() {
	echo 检查用户
}

check_num() {
	echo 检查数字
}
```

## 1.1 自动运行脚本

书写Hello world脚本并在用户登录的时候自动运行。  

```shell
#!/bin/bash
echo "Hello World"
```

追加到`/etc/bashrc`的最后面

```shell
[root@mn01[ ~]#tail -n 1 /etc/bashrc 
bash /server/scripts/devops-shell/shell-108/02_hello.sh
```

测试

```shell
[root@mn01[ ~]#su root
主机名：mn01
ip地址：10.0.0.61 172.16.1.61 
总内存：1.9G
可用内存：1.6G
系统负载：0.00, 0.01, 0.05
Hello World
```

## 1.2 备份日志

备份日志/var/log下面所有内容到/backup

```shell
#!/bin/bash
####################################################
##########
# File Name:02.backup.sh
# Version:V1.0
# Author:oldboy lidao996
# Organization:www.oldboyedu.com
# Desc:
####################################################
##########
#1.vars
time=`date +%F`
filename=log-${time}.tar.gz

#2.打包日志文件，备份到/backup中
tar zcf /backup/${filename} /var/log/
```

备份用户手动输入的目录

```shell
# 可以修改方向:
#1.备份任意多的目录,文件
#使用方法：sh 02.backup.sh /etc/ /var/log/ /oldboy/oldboy.txt
tar zcf /backup/${filename} $@
```

## 1.3 一键部署LNMP（RPM方式）

RPM包方式

```shell
#!/bin/bash

#1. vars
# 准备nginx,php,mariadb的rpm包
rpmdir=/server/tools/lnmp/

#2.yum安装
yum localinstall -y /server/tools/lnmp/*.rpm
#如果不是软件包yum方式
#配置ngx和phpyum源
yum install -y nginx
yum install -y phpxxxxxxxx
yum install -y mariadb
```



## 1.4  一键部署LNMP（yum方式）

```shell
#!/bin/bash
yum install -y httpd
yum install -y phpxxxxxxxx
yum install -y mariadb
```



## 1.5 一键部署redis

yum安装方式

```shell
#!/bin/bash

yum install -y redis
```



## 1.6 远程其他主机安装httpd软件  

使用sshpass工具自动交互密码远程其他主机安装httpd软件  

```shell
#!/bin/bash

# 导入通用函数库
. diy_func.sh

#1.检查sshpass是否安装
check_cmd() {
	cmd=$1
	rpm=$2
	which $cmd &>/dev/null || {
		redecho "$cmd 命令不存在"
		yum install -y $rpm &>/dev/null
	}
}

#2. 连接并安装软件
ssh_cmd() {
	pass=1
	sshpass -p$pass ssh root@10.0.0.61 "yum install -y httpd"
	check_return_value sshpass
}

# 3.main函数
main() {
	check_cmd sshpass sshpass
	#check_cmd ifconfig net-tools
	ssh_cmd
}
# 执行main
main
```

## 1.7 查看有多少远程IP在连接本机

```shell
#!/bin/bash

#检查当前有多少人远程连接这个主机的22端口.
ss -ant |grep -i ESTAB|awk '$4~/:22$/' |wc -l

#有多少ip连接本地的任意端口
ss -ant |grep -i ESTAB |wc -l
ss -ant |grep -ic ESTAB


#统计登录的用户数
w命令统计登录用户数量
```



## 1.8 统计当前 Linux 系统中可以登录计算机的账户有多少个  

```shell
#!/bin/bash
##############################################################
# File Name:08_login_user.sh
# Version:V1.0
# Author:Haris Gong
# Organization:gsproj.github.io
# Desc:
##############################################################


#分析:
#可以登录:
#1. 命令解释器是/etc/passwd /bin/bash
#2. 密码 /etc/shadow 第2列不是*号,也不是包含Վʰ
#思路1.
##先取出1个文件的内容(满足条件的)
##然后在另外一个文件过滤并判断.

. diy_func.sh

#1.过滤/etc/passwd命令解释器是/bin/bash的用户
check_login_etc_passwd() {
name_list=`grep '/bin/bash' /etc/passwd|awk -F: '{print $1}'`
}
#2.根据这个用户去/etc/shadow里面过滤,如果第2列是非*或!就可以登录.
check_login_etc_shadow() {
	i=0
	for name in $name_list
	do
		login=`grep -w "${name}" /etc/shadow |awk -F: '{ if($2!~/[*!]/) print 0 ;else print 1 }'`
		if [ $login -eq 0 ];then
			greenecho "$name 用户可以登录"
			let i++
		fi
	done
	blueecho "可以远程登录系统的用户数量是:$i"
}

#3.main
main() {
	check_login_etc_passwd
	check_login_etc_shadow
}
main
```

测试

![image-20240522163741237](../../../img/image-20240522163741237.png)



## 1.9 统计/var/log 有多少个文件

需要显示这些文件名  

```shell
#!/bin/bash

#1.find 看所有层
find /var/log -type f | wc -l	# 显示文件个数
find /var/log -type f	# 显示文件名
```



## 1.10 自动化部署Tengine源码包

```shell
#1. ./configure
./configure --prefix=/app/tools/tengine-2.3.3/ --add-module=负载均衡状态检查模块

#2.编译 
make -j `nproc`

#3、安装
make install
```

