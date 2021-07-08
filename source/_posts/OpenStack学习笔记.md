---
title: OpenStack学习笔记
categories:
- OpenStack
tags:
- OpenStack
- 学习笔记
---

# **<font color=green>OpenStack笔记</font>**

​	OpenStack实现的是云计算IAAS

## <font color=blue>一、服务架构发展</font>

### 1.1 MVC架构

​	业务不拆分，一个服务挂，则所有的全挂

```shell
首页 www.jd.com/index.html
秒杀 www.jd.com/miaosha/index.html
优惠券 www.jd.com/juan/index.html
```

### 1.2 SOA架构（千万级）

​	业务拆分，每一个功能都拆分成一个独立的web服务，每个独立的web服务，都至少拥有一个集群

```she
首页 www.jd.com/index.html
秒杀 miaosha.jd.com/index.html
优惠券 juan.jd.com/index.html
```

### 1.3 微服务架构（亿级）

阿里开源dubbo

Spring Boot

自动化代码上线：Jekins + gilab ci

自动化代码质量检查：sonarqube



## <font color=blue>二、搭建OpenStack</font>

​	本流程为手动安装M版，脚本安装可以参考

```shell
https://my.oschina.net/u/4367225/blog/4255750
```

OpenStack的结构介绍：

>Nova -- 提供VM虚拟化支持 8774
>
>Glance -- 提供镜像 9292
>
>Clinder -- 存储支持 8776
>
>Neutron -- 网络支持 9696
>
>Cellometer --  监控计费 
>
>KeyStone -- 登录认证
>
>Horizon -- 网页UI，dashboard
>
>Heat -- 部署编排，批量建虚拟机
>
>Switft -- 对象存储（不是传统的文件夹存放，而是用数据库记录已上传的文件信息，当有文件上传，先查询数据库中是否有该文件的md5值，如果有，则不用重新上传，给个链接就是 --- 百度云盘）

![image-20210615135957367](C:\Users\fr724\AppData\Roaming\Typora\typora-user-images\image-20210615135957367.png)

### 2.1 虚拟机准备

虚拟机规划

```shell
# 系统：CentOS7.4
controller: 内存3G, CPU开启虚拟化	10.0.0.11
compute1: 内存1G，CPU开启虚拟化	    10.0.0.31
# 修改主机名，IP地址，host解析，测试ping百度
```

配置本地M版yum源

```shell
# 1、资源准备
mount /dev/cdrom /mnt # 追加到/etc/rc.local,自动挂载
解压openstack_rpm.tar.gz到/opt/repo

# 2、编辑repo文件
vim /etc/yum.repo.d/local/repo

#### 内容
[local]
name=local
baseurl=file:///mnt
gpgcheck=0

[openstack]
name=openstack
baseurl=file:///opt/repo
gpgcheck=0
####

# 3、更新yum源
yum makecache
yum repolist
```



### 2.2 基础服务安装

#### 2.2.1 NTP时间同步

controller与阿里NTP服务器同步

```shell
 vim /etc/chrony.conf
 server ntp6.aliyun.com # 3行
 allow 10.0.0.0/24 # 24行
 systemctl restart chronyd
```

computer与controller同步

```shell
vim /etc/chrony.conf
server 10.0.0.11 iburst # 3行
systemctl restart chronyd
```

#### 2.2.2 扩展-公网安装O版OpenStack的方法介绍（跳过该步骤）

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
yum list | grep openstack
yum install centos-release-openstack-ocata.noarch -y # 安装O版
```

#### 2.2.3 安装OpenStack客户端openstack-selinux （所有节点执行）

```shell
yum install python-openstackclient openstack-selinux -y
```

#### 2.2.4 安装和配置mariadb (仅控制节点执行)

安装

```shell
yum install mariadb mariadb-server python2-PyMySQL -y
```

配置

```shell
vim /etc/my.cnf.d/openstack.cnf

#----------------
[mysqld]	
bind-address = 10.0.0.11	# 监听地址
default-storage-engine = innodb # 默认存储引擎
innodb_file_per_table  # 独立表空间文件
max_connections = 4096	# 最大连接数
collation-server = utf8_general_ci	# 默认字符集utf8
character-set-server = utf8
#-----------------

# 启动服务
systemctl start mariadb
systemctl enable mariadb

# 数据库安全初始化,保障数据库安全性，如果不执行，同步数据库表会报错
mysql_secure_installation
回车 n y y y y
```

#### 2.2.5 消息队列配置(仅控制节点执行)

```shell
# 安装rabbitmq
yum install rabbitmq-server -y
# 启动服务
systemctl start rabbitmq-server
systemctl enable rabbitmq-server
# 添加用户
rabbitmqctl add_user openstack RABBIT_PASS
# 设置用户权限（读、写、执行）
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
rabbitmq-plugins enable rabbitmq_management
# 查看是否开启15672端口
netstat -lntup
# 浏览器登录rabbitmq
http://10.0.0.11:15672
默认用户名和密码：guest
```

#### 2.2.6 缓存系统配置memcache（仅控制节点执行）

```shell
# 安装
yum install memcached python-memched -y
# 配置
sed -i 's#127.0.0.1#10.0.0.11#g' /etc/sysconfig/memcached
# 启动服务
systemctl restart memcached
systemctl enable memcached
# 查询端口11211是否已监听，默认使用该端口
```

### 2.3 安装keystone认证服务(仅控制节点执行)

#### 2.3.1 Keystone功能介绍

```shell
1、认证管理：
	账户密码
2、授权管理
3、服务目录：
	跟电话本一样，keystone上可以查询到glance、nova等服务的地址端口等信息，每一个新加的服务都需要在keystone上注册
```

#### 2.3.2 OpenStack服务器安装的通用步骤

```shell
1、创库授权
2、在Keystone创建用户，关联角色
3、在keystone创建服务，注册api
4、安装服务相关的软件包
5、修改配置
	数据库的连接
	keystone认证授权信息
	rabbitmq连接信息
	其他配置
6、同步数据库，创建表
7、启动服务
```

#### 2.3.3 安装步骤

1、创库授权

```mysql
# 登录mysql
$ mysql -u root -p
# 创建keystone数据库
CREATE DATABASE keystone;
# 对``keystone``数据库授予恰当的权限
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'KEYSTONE_DBPASS';
```

2、安装keystone相关软件包

```shell
yum install openstack-keystone httpd mod_wsgi -y
```

3、修改配置文件

```shell
# 备份原配置文件
\cp /etc/keystone/keystone.conf{,.bak}
# 去除配置文件中的空格行和注释行
grep -Ev '^$|#' /etc/keystone/keystone.conf.bak > /etc/keystone/keystone.conf

# 安装自动配置工具
yum install openstack-utils -y
# 使用工具设置（也可以直接修改文件）修改项 参数 = 值
openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token ADMIN_TOKEN
openstack-config --set /etc/keystone/keystone.conf database connection  mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone # 注意hostname--controller
openstack-config --set /etc/keystone/keystone.conf token provider fernet
# 校验
md5sum /etc/keystone/keystone.conf
d5acb3db852fe3f247f4f872b051b7a9 

# 同步数据库
su -s /bin/sh -c "keystone-manage db_sync" keystone
# 查询是否生成表
mysql keystone -e "show tables;"

# 初始化fernet
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# 验证
/etc/keystone/fernet-keys已创建

# 配置httpd
echo "ServerName controller" >> /etc/httpd/conf/httpd.conf

# 创建wsgi配置文件
vim /etc/httpd/conf.d/wsgi-keystone.conf
###内容
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
# 校验
md5sum /etc/httpd/conf.d/wsgi-keystone.conf
8f051eb53577f67356ed03e4550315c2 

# 启动httpd
systemctl enable httpd
systemctl start httpd

# 创建服务和注册api
export OS_TOKEN=ADMIN_TOKEN
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
openstack service create --name keystone --description "OpenStack Identity" identity
openstack endpoint create --region RegionOne identity public http://controller:5000/v3
openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
openstack endpoint create --region RegionOne identity admin http://controller:35357/v3

# 创建域、项目（租户）、用户和角色
openstack domain create --description "Default Domain" default
openstack project create --domain default --description "Admin Project" admin
openstack user create --domain default --password ADMIN_PASS admin
openstack role create admin

# 关联项目，用户，角色
openstack role add --project admin --user admin admin
# 在admin项目上，给admin用户赋予admin角色
openstack project create --domain default --description "Service Project" service

# 创建环境变量脚本
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_IMAGE_API_VERSION=2
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_URL=http://controller:35357/v3

# 作为admin用户请求认证令牌
openstack token issue  

# 如果不设置环境变量可以通过传入参数的方式申请
openstack --os-auth-url http://controller:35357/v3   --os-project-domain-name default --os-user-domain-name default   --os-project-name admin --os-username admin --os-password ADMIN_PASS token issue

# 查看用户列表
openstack user list

# 查看endpoint列表
openstack endpoint list
```

### 2.4 安装glance镜像服务

​	镜像服务 (glance) 允许用户发现、注册和获取虚拟机镜像。

#### 2.4.1 安装步骤

```shell
# 数据库创库授权
mysql >>
CREATE DATABASE glance
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'GLANCE_DBPASS';
  
 # 在keystone创建glance用户关联角色
 openstack user create --domain default --password GLANCE_PASS glance
 openstack role add --project service --user glance admin
 
 # 在keystone上创建服务和注册api
openstack service create --name glance   --description "OpenStack Image" image
openstack endpoint create --region RegionOne \
  image public http://controller:9292
openstack endpoint create --region RegionOne \
  image internal http://controller:9292
openstack endpoint create --region RegionOne \
  image admin http://controller:9292
  
# 查看已创建的信息
openstack role assignment list
openstack role list
openstack project list
openstack user list （要有glance用户）

# mysql中验证表是否已创建
[root@controller ~]# mysql keystone -e "show tables;" | grep user
federated_user
local_user
user
user_group_membership
[root@controller ~]# mysql keystone -e "show tables;" | grep project
project
project_endpoint
project_endpoint_group

# 安装服务相应软件包
yum install openstack-glance -y

# 修改相应的配置文件--api
cp /etc/glance/glance-api.conf{,.bak}
grep '^[a-Z\[]' /etc/glance/glance-api.conf.bak > /etc/glance/glance-api.conf
openstack-config --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password GLANCE_PASS
openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/

# 修改相应的配置文件--registry
cp /etc/glance/glance-registry.conf{,.bak}
grep '^[a-Z\[]' /etc/glance/glance-registry.conf.bak > /etc/glance/glance-registry.conf
openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken password GLANCE_PASS
openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone

# md5值验证
md5sum /etc/glance/glance-api.conf
3e1a4234c133eda11b413788e001cba3  /etc/glance/glance-api.conf
md5sum /etc/glance/glance-registry.conf
46acabd81a65b924256f56fe34d90b8f  /etc/glance/glance-registry.conf

# 写入镜像服务数据库
su -s /bin/sh -c "glance-manage db_sync" glance # 会有Warning不用在意
# 验证
mysql glance -e "show tables;"

# 启动服务
systemctl enable openstack-glance-api.service   openstack-glance-registry.service
systemctl start openstack-glance-api.service   openstack-glance-registry.service
```

#### 2.4.2 上传镜像测试

```shell
# 错误日志查看
/var/log/glance

# 确保已获取token令牌
openstack token issue

# 下载测试镜像
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

# 上传测试镜像
openstack image create "cirros" \
--file cirros-0.3.4-x86_64-disk.img \
--disk-format qcow2 --container-format bare \
--public

# 上传文件确认
ls /var/lib/glance/images/ (在/etc/glance/glance-api.conf中设置)
mysql glance -e "show tables;" | grep image
```

### 2.5 安装nova计算服务

```shell
nova-api -- 接受并相应所有的计算服务请求，管理云主机生命周期
nova-compute（多个）-- 真正管理虚拟机(nova-compute调用libvirt)
nova-scheduler -- nova 调度器（挑选最合适的nova-compute）
nova-conductor -- 帮助nova-compute连接数据库
nova-network --  早期版本管理虚拟机的网络（已弃用，改用neutron，留着为了方便兼容早期版本）
nova-consoleauth和nova-novncproxy -- web版的vnc来直接操作云主机
novnproxy -- web版vnc客户端
nova-api-metadata -- 接受来自虚拟机发送的元数据请求（配合neutron-metadata-agent实现虚拟机定制化）
```

#### 2.5.1 控制节点--安装步骤

```shell
# 创库授权
mysql >>
CREATE DATABASE nova_api;
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
  
# 在keystone创建用户nova
openstack user create --domain default   --password NOVA_PASS nova
# 给Nova用户添加admin角色
openstack role add --project service --user nova admin

# 在keystone上创建服务和注册api
openstack service create --name nova   --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne   compute public http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne   compute internal http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne   compute admin http://controller:8774/v2.1/%\(tenant_id\)s

# 安装相应软件包
yum install -y openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler
  
# 修改配置文件
cp /etc/nova/nova.conf{,.bak}
grep -Ev '^$|#' /etc/nova/nova.conf.bak > /etc/nova/nova.conf

openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.0.0.11
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
openstack-config --set /etc/nova/nova.conf database connection mysql+pymysql://nova:NOVA_DBPASS@controller/nova
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address '$my_ip'

# 校验
md5sum /etc/nova/nova.conf
47ded61fdd1a79ab91bdb37ce59ef192  /etc/nova/nova.conf

# 同步数据库
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova

# 验证
mysql nova_api -e "show tables;"
mysql nova -e "show tables;"

# 启动服务
systemctl enable openstack-nova-api.service \
openstack-nova-consoleauth.service openstack-nova-scheduler.service \
openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl restart openstack-nova-api.service \
openstack-nova-consoleauth.service openstack-nova-scheduler.service \
openstack-nova-conductor.service openstack-nova-novncproxy.service

# 验证
[root@controller ~]# nova service-list
+----+------------------+------------+----------+---------+-------+------------+-----------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated_at | Disabled Reason |
+----+------------------+------------+----------+---------+-------+------------+-----------------+
| 1  | nova-conductor   | controller | internal | enabled | down  | -          | -               |
| 4  | nova-scheduler   | controller | internal | enabled | down  | -          | -               |
| 5  | nova-consoleauth | controller | internal | enabled | down  | -          | -               |
+----+------------------+------------+----------+---------+-------+------------+-----------------+

# novncproxy 怎么检测起来没有？
[root@controller ~]# netstat -lntup | grep 6080
tcp        0      0 0.0.0.0:6080            0.0.0.0:*               LISTEN      8719/python2
[root@controller ~]# ps -ef | grep 8719
nova       8719      1  0 21:53 ?        00:00:01 /usr/bin/python2 /usr/bin/nova-novncproxy --web /usr/share/novnc/
```

#### 2.5.1 计算节点--安装步骤

```shell
# 安装软件包
yum install -y openstack-nova-compute openstack-utils

# 修改配置文件
cp /etc/nova/nova.conf{,.bak}
grep -Ev '^$|#' /etc/nova/nova.conf.bak > /etc/nova/nova.conf

openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.0.0.31 # 注意ip
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/nova/nova.conf vnc enabled  True
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 0.0.0.0
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address '$my_ip'
openstack-config --set /etc/nova/nova.conf vnc novncproxy_base_url http://controller:6080/vnc_auto.html

# 校验
md5sum /etc/nova/nova.conf
45cab6030a9ab82761e9f697d6d79e14  /etc/nova/nova.conf

# 启动服务
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl restart libvirtd.service openstack-nova-compute.service

# 服务启动错误排查
参考博客：https://www.codeleading.com/article/89785382846/
cat /etc/nova/nova.conf # compute1日志查看
报错 nova AccessRefused: (0, 0): (403) ACCESS_REFUSED
处理步骤：
在controller
cat /var/log/rabbitmq/rabbit@controller.log # 发现报错AMQPLAIN login refused: user 'openstack' - invalid credentials 无效凭证
rabbitmqctl list_users # 确认是否还有openstack用户
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
systemctl restart rabbitmq-server.service # 重新创用户，并重启服务
```

### 2.6 安装neutron网络服务

neutron-server -- 端口9696，api接受和响应外部的网络管理请求

neutron-linuxbridge-agent --  负责创建桥接网卡

neutron-dhcp-agent -- 负责分配ip

neutron-metadata-agent --  配合nova-metadata-api实现虚拟机的定制化操作

L3-agent -- 实现三层网络vxlan（网络层）

#### 2.6.1 控制节点--安装步骤

```shell
# 创库授权
Mysql >>
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'NEUTRON_DBPASS';
  
# 在keystone创建用户neutron
openstack user create --domain default   --password NEUTRON_PASS neutron
# 给neutron用户添加admin角色
openstack role add --project service --user neutron admin

# 在keystone上创建服务和注册api
openstack service create --name neutron   --description "OpenStack Networking" network
openstack endpoint create --region RegionOne   network public http://controller:9696
openstack endpoint create --region RegionOne   network internal http://controller:9696
openstack endpoint create --region RegionOne   network admin http://controller:9696

# 网络配置--公共网络
cp /etc/neutron/neutron.conf{,.bak}
grep -Ev "^$|#" /etc/neutron/neutron.conf.bak  > /etc/neutron/neutron.conf
>>>>>
openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:NEUTRON_DBPASS@controller/neutron
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password NEUTRON_PASS
openstack-config --set /etc/neutron/neutron.conf nova auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf nova auth_type password
openstack-config --set /etc/neutron/neutron.conf nova project_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova user_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova region_name RegionOne
openstack-config --set /etc/neutron/neutron.conf nova project_name service
openstack-config --set /etc/neutron/neutron.conf nova username nova
openstack-config --set /etc/neutron/neutron.conf nova password NOVA_PASS
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
>>>>>

cp /etc/neutron/plugins/ml2/ml2_conf.ini{,.bak}
grep -Ev "^$|#" /etc/neutron/plugins/ml2/ml2_conf.ini.bak > /etc/neutron/plugins/ml2/ml2_conf.ini
>>>>>
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
>>>>>

cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini{,.bak}
grep -Ev "^$|#" /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak > /etc/neutron/plugins/ml2/linuxbridge_agent.ini
>>>>>
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens33  # 要修改网络接口名
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
>>>>>

cp /etc/neutron/dhcp_agent.ini{,.bak}
grep -Ev "^$|#" /etc/neutron/dhcp_agent.ini.bak > /etc/neutron/dhcp_agent.ini
>>>>>
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT  dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT  enable_isolated_metadata True
>>>>>

cp /etc/neutron/metadata_agent.ini{,.bak}
grep -Ev "^$|#" /etc/neutron/metadata_agent.ini.bak > /etc/neutron/metadata_agent.ini
>>>>>
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET
>>>>>

# 再次修改nova配置文件，添加neutron服务
openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696
openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf neutron auth_type password
openstack-config --set /etc/nova/nova.conf neutron project_domain_name default
openstack-config --set /etc/nova/nova.conf neutron user_domain_name default
openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
openstack-config --set /etc/nova/nova.conf neutron project_name service
openstack-config --set /etc/nova/nova.conf neutron username neutron
openstack-config --set /etc/nova/nova.conf neutron password NEUTRON_PASS
openstack-config --set /etc/nova/nova.conf neutron service_metadata_proxy True
openstack-config --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret METADATA_SECRET
# 校验文件
[root@controller ~]# md5sum /etc/nova/nova.conf
6334f359655efdbcf083b812ab94efc1  /etc/nova/nova.conf

# 网络服务初始化脚本需要一个超链接
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini

# 同步数据库
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
  
# 检查数据库
mysql neutron -e "show tables;"

# 重启Nova服务
systemctl restart openstack-nova-api.service

# 启动Neutron服务
systemctl enable neutron-server.service   neutron-linuxbridge-agent.service neutron-dhcp-agent.service   neutron-metadata-agent.service
systemctl start neutron-server.service   neutron-linuxbridge-agent.service neutron-dhcp-agent.service   neutron-metadata-agent.service

# 查看服务有没有起来
neutron agent-list
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| 0d68dc27-f2b8-4cae-9c30-cddd126076b4 | Linux bridge agent | controller |                   | :-)   | True           | neutron-linuxbridge-agent |
| 688cc47d-424d-4243-ae2b-b1c4b298a2a8 | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-agent    |
| c4c5e360-ee02-4b9b-a1a5-59a0d0d088b0 | DHCP agent         | controller | nova              | :-)   | True           | neutron-dhcp-agent        |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
```

#### 2.6.2 计算节点--安装步骤

```shell
# 安装组件
yum install -y openstack-neutron-linuxbridge ebtables ipset

# 网络配置--公共网络
cp /etc/neutron/neutron.conf{,.bak}
grep -Ev "^$|#" /etc/neutron/neutron.conf.bak  > /etc/neutron/neutron.conf
>>>>>>
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password RABBIT_PASS
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password NEUTRON_PASS
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
>>>>>>
# md5校验
md5sum /etc/nova/nova.conf
328cd5f0745e26a420e828b0dfc2934e  /etc/nova/nova.conf

cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini{,.bak}
grep -Ev "^$|#" /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak > /etc/neutron/plugins/ml2/linuxbridge_agent.ini
>>>>>>
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens33  # 要修改网络接口名
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan False
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
>>>>>>

# 再次修改nova配置文件，添加neutron配置
openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696
openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf neutron auth_type password
openstack-config --set /etc/nova/nova.conf neutron project_domain_name default
openstack-config --set /etc/nova/nova.conf neutron user_domain_name default
openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
openstack-config --set /etc/nova/nova.conf neutron project_name service
openstack-config --set /etc/nova/nova.conf neutron username neutron
openstack-config --set /etc/nova/nova.conf neutron password NEUTRON_PASS

# 重启nova服务
systemctl restart openstack-nova-compute.service

# 启动linuxbridge代理服务
systemctl enable neutron-linuxbridge-agent.service
systemctl restart neutron-linuxbridge-agent.service

# 查看是否配置成功
controller执行 >> neutron agent-list
配置正确会多出来一个
Linux bridge agent | compute1   |                   | :-)   | True 

# 查看计算资源
openstack compute service list
```

### 2.7 安装horizon （Dashboard）web界面

#### 2.7.1 安装步骤（控制节点)

```shell
# 安装组件包
yum install -y openstack-dashboard

# 修改配置文件
vim /etc/openstack-dashboard/local_settings
>>>>
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*', ]
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_router': False,
    'enable_quotas': False,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,
    'enable_fip_topology_check': False,
}
TIME_ZONE = "Asia/Shanghai"
>>>>

# 解决不能进入页面的BUG
vim /etc/httpd/conf.d/openstack-dashboard.conf
添加一行 >> WSGIAppicationGroup %{GLOBAL}

# 重启服务
systemctl restart httpd.service memcached.service

# 登录界面
https://10.0.0.11/dashboard
域:default
用户名:admin
密码:ADMIN_PASS

# 扩展：查看文件输入那个rpm包
rpm -qf /etc/openstack-dashboard/local_settings
```

#### 2.7.2 Dashboard报错解决

```shell
# 问题1--Invalid service catalog service: image
发生原因：
	openstack service list 存在两个 glance
解决方法：
    openstack service delete c12c125edc2041e3aaf2f250442162c6
    openstack service delete 6a11431b95bc44d1bd1e9371c0faa16b # 两个glance都删掉
    重复2.4.1在keystone上创建glance服务和注册api
    再重启服务systemctl restart httpd.service memcached.service
```

### 2.8 启动一个云主机

### 2.8.1 创建步骤

```shell
# 创建网络
neutron net-create --shared --provider:physical_network provider \
  --provider:network_type flat gs
  
# 在网络中创建一个子网
neutron subnet-create --name gs2 \
  --allocation-pool start=10.0.0.101,end=10.0.0.250 \
  --dns-nameserver 223.5.5.5 --gateway 10.0.0.2 \
  gs 10.0.0.0/24
  
# 创建云主机硬件配置方案（规格）
openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano

# 创建密钥键值对
ssh-keygen -q -N "" -f ~/.ssh/id_rsa
openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
# 验证公钥的添加
openstack keypair list

# 添加安全组规则
openstack security group rule create --proto icmp default
openstack security group rule create --proto tcp --dst-port 22 default

# 在公有网络启动一个实例
openstack flavor list # 查看可用的配置规格
openstack image list # 查看可用镜像
openstack network list # 查看可用网络
openstack security group list # 查看已设置的安全组

openstack server create --flavor 规格名/ID --image 镜像名/ID \
  --nic net-id=网络ID --security-group default \
  --key-name mykey 实例名称
示例：
openstack server create --flavor m1.tiny --image cirros \
--nic net-id=cb032582-893b-414d-b78d-c89c6548612d --security-group default \
--key-name mykey my-instance
```

### 2.8.2 创建云主机问题解决

```shell
# 1、启动云主机时，No valid host was found
【计算节点中】
查看云主机创建日志：/var/log/nova/nova-compute.log，里面有记录问题原因是CPU feature spec-ctrl not found
修改/usr/share/libvirt/cpu_map.xml，将和spec-ctrl相关的特性删除
然后重启服务
systemctl restart libvirtd openstack-nova-compute
参考博客：https://www.cnblogs.com/laolieren/p/solve_openstack_create_instance_error.html

# 2、云主机控制台seabios -- Booting from Hard Disk错误
【计算节点中】
vim /etc/nova/nova.conf
>>>>
[libvirt]
virt_type = qemu
cpu_mode = none
>>>>
systemctl restart libvirtd openstack-nova-compute
重启云主机
```







