---
title: openvpn 使用手册
date: 2022-07-11 9:29:52
categories:
- 杂记
tags:
---


# openvpn 使用手册

此手册的目的是使读者快速上手将openvpn使用起来，满足大部分需求

## 一、环境

硬件|操作系统|软件
----|-------|-----
ARM64 ft2000+|centos7.5|easy-rsa3.0.6 openvpn2.4.7

## 二、准备安装源

```bash
[root@openvpn ~]# cat /etc/yum.repos.d/CentOS-Base.repo
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.aliyun.com/centos-altarch/$releasever/os/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.aliyun.com/centos-altarch/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.aliyun.com/centos-altarch/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
enabled=1

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.aliyun.com/centos-altarch/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

```shell
[root@openvpn ~]# cat /etc/yum.repos.d/epel.repo
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/7/$basearch/debug
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```

## 三、安装软件包

```bash
[root@openvpn ~]# yum install easy-rsa openvpn -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * epel: mirrors.yun-idc.com
Package easy-rsa-3.0.6-1.el7.noarch already installed and latest version
Package openvpn-2.4.7-1.el7.aarch64 already installed and latest version
Nothing to do
[root@openvpn ~]#
```

## 四、配置easy-rsa 3.0.6

安装完easy-rsa 3.0.6 后，默认会生成 easyrsa 脚本文件和 vars.example 环境变量文件，这两件文件是核心文件，下面会用到

找到这两个文件所在的目录，复制到相应的位置，然后根据自己的需求配置vars

```bash
[root@openvpn ~]# find /usr/ -name easyrsa -o -name vars.example
/usr/share/doc/easy-rsa-3.0.6/vars.example
/usr/share/easy-rsa/3.0.6/easyrsa
[root@openvpn ~]# cp -r /usr/share/easy-rsa/3.0.6/ /etc/openvpn/3.0.6/
[root@openvpn ~]# cp /usr/share/doc/easy-rsa-3.0.6/vars.example /etc/openvpn/3.0.6/vars
[root@openvpn ~]# grep -v "^#\|^$" /etc/openvpn/3.0.6/vars
if [ -z "$EASYRSA_CALLER" ]; then
    echo "You appear to be sourcing an Easy-RSA 'vars' file." >&2
    echo "This is no longer necessary and is disallowed. See the section called" >&2
    echo "'How to use this file' near the top comments for more details." >&2
    return 1
fi
set_var EASYRSA_REQ_COUNTRY	"CN"
set_var EASYRSA_REQ_PROVINCE	"HuNan"
set_var EASYRSA_REQ_CITY	"ChangSha"
set_var EASYRSA_REQ_ORG	"Greatwall Corporation"
set_var EASYRSA_REQ_EMAIL	"yuliangliang@greatwall.com.cn"
set_var EASYRSA_REQ_OU		"Product Department"
[root@openvpn ~]# 
```

### easyrsa 命令使用

```bash
// 查看整体帮助
[root@openvpn 3.0.6]# ./easyrsa -h

// 查看某个子命令的详细帮助
[root@openvpn 3.0.6]# ./easyrsa help import-req

Note: using Easy-RSA configuration from: ./vars

  import-req <request_file_path> <short_basename>
      Import a certificate request from a file

      This will copy the specified file into the reqs/ dir in
      preparation for signing.
      The <short_basename> is the filename base to create.

      Example usage:
        import-req /some/where/bob_request.req bob
[root@openvpn 3.0.6]#

// 查看可以用到的 options
[root@openvpn 3.0.6]# ./easyrsa help options
```

### 创建 PKI 和 CA

```bash
[root@openvpn ~]# cd /etc/openvpn/3.0.6/
[root@openvpn 3.0.6]# ./easyrsa init-pki
[root@openvpn 3.0.6]# ls
easyrsa  openssl-easyrsa.cnf  pki  vars  x509-types
[root@openvpn 3.0.6]# ./easyrsa build-ca nopass (配置好了vars直接回车即可)
```

### 创建服务端证书

```bash
[root@openvpn 3.0.6]# ./easyrsa gen-req server nopass (回车即可)
```

### 签约服务端证书

```bash
[root@openvpn 3.0.6]# ./easyrsa sign server server
```

### 创建 Diffie-Hellman

```bash
[root@openvpn 3.0.6]# ./easyrsa gen-dh
```

### 创建客户端证书

```bash
[root@openvpn 3.0.6]# ./easyrsa gen-req jiajia nopass
```

### 签约客户端证书

```bash
[root@openvpn 3.0.6]# ./easyrsa sign client jiajia
```

注意: 注意这里的客户端证书用到的pki目录跟服务器端的pki目录是一样的，所以可以直接 sign签署，如果客户端重新生成了pki，那每次生成的证书请求文件都需要用 ./easyrsa import-req $PATH/jiajia.req jiajia 导入并取一个短名字jiajia，然后执行 ./easyrsa sign client jiajia；参见 ./easy-rsa help import-req

### 生成ta.key文件（防DDos攻击、UDP淹没等恶意攻击）

```bash
[root@openvpn 3.0.6]# openvpn --genkey --secret ta.key
```

### 整理证书

```bash
[root@openvpn 3.0.6]# pwd
/etc/openvpn/3.0.6
[root@compute3 3.0.6]# ll pki/private/
total 8
-rw-------. 1 root root 1679 Jun  9 21:05 ca.key
-rw-------. 1 root root 1708 Jun  9 21:06 server.key
[root@compute3 3.0.6]#
[root@compute3 3.0.6]# ll pki/issued/
total 8
-rw-------. 1 root root 4552 Jun  9 21:18 server.crt
[root@compute3 3.0.6]#
[root@compute3 3.0.6]# ll pki/ca.crt 
-rw-------. 1 root root 1172 Jun  9 21:05 pki/ca.crt
[root@compute3 3.0.6]# 
[root@compute3 3.0.6]# ll pki/dh.pem 
-rw-------. 1 root root 424 Jun  9 21:10 pki/dh.pem
[root@compute3 3.0.6]#
[root@compute3 3.0.6]# cp ta.key pki/
[root@compute3 3.0.6]# ll pki/ta.key 
-rw-------. 1 root root 636 Jun  9 21:31 pki/ta.key
[root@compute3 3.0.6]# 
```

以上配置文件将会在服务器端和端户端配置文件中指出

### 服务器端配置文件

以11楼跳板机为例（其实就是当路由器用）：

>eno1：10.60.30.145/24  网关:10.60.30.250 (连接外网)
>
>eno2: 10.10.10.253  网关: 10.10.10.254 
>
>enp94s0: 10.10.11.253 网关：10.10.11.254

```bash
[root@openvpn 3.0.6]# grep -vE "^#|^$" /etc/openvpn/server.conf （该文件默认没有，需要自己创建）
local 10.60.30.145 # 服务器本机连接外网的网卡IP
port 1194 # 默认端口号，需要防火墙放通，做端口映射
proto tcp
dev tun
ca /etc/openvpn/3.0.6/pki/ca.crt
cert /etc/openvpn/3.0.6/pki/issued/server.crt
key /etc/openvpn/3.0.6/pki/private/server.key
dh /etc/openvpn/3.0.6/pki/dh.pem
server 10.8.0.0 255.255.255.0 # vpn地址池，不用改
ifconfig-pool-persist ipp.txt
push "route 10.10.0.0 255.255.0.0" # 连接内网的路由，这边是10.10.10和10.10.11两个网段
keepalive 10 120
tls-auth /etc/openvpn/3.0.6/pki/ta.key 0
cipher AES-256-CBC
comp-lzo
persist-key
persist-tun
status openvpn-status.log
verb 3
reneg-sec 0

# 注意，这个文件用来记录被吊销的证书，阻止被吊销的客户再次登陆
# 默认没有这个文件，必须执行 ./easyrsa gen-crl 更新生成，否则服务启不来
crl-verify /etc/openvpn/3.0.6/pki/crl.pem
[root@openvpn 3.0.6]#
```

或者可以参考这一份配置文件

路由器（公网IP + 端口映射）--- 内网服务器（15.1.10.32，部署OpenVPN服务1194端口）

实现的效果：

- Windows客户端连接后，将获取IP，10.8.0.6
- 测试可以与openvpn服务器ping通，ping 10.8.0.1
- 测试可以与OpenVPN服务器内网IP互通，ping 15.1.10.32

```shell
port 1194
# 改成tcp，默认使用udp，如果使用HTTP Proxy，必须使用tcp协议
proto tcp
dev tun
# # 路径前面加keys，全路径为/etc/openvpn/keys/ca.crt
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key  # This file should be kept secret
dh /etc/openvpn/easy-rsa/pki/dh.pem

# # 默认虚拟局域网网段，不要和实际的局域网冲突即可
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
# # VPN服务器所在的内网的网段
push "route 15.1.10.0 255.255.255.0"
# # 让客户端所有的流量都通过VPN转发(全局模式)
# push "redirect-gateway def1 bypass-dhcp"
# # 可以让客户端之间相互访问直接通过openvpn程序转发
# client-to-client
# # 如果客户端都使用相同的证书和密钥连接VPN，一定要打开这个选项，否则每个证书只允许一个人连接VPN
# duplicate-cn
keepalive 10 120
tls-auth /etc/openvpn/easy-rsa/pki/ta.key 0
comp-lzo
persist-key
persist-tun
# # OpenVPN的状态日志，默认为/etc/openvpn/openvpn-status.log
status openvpn-status.log
# # OpenVPN的运行日志，默认为/etc/openvpn/openvpn.log 
log-append openvpn.log
# # 改成verb 5可以多查看一些调试信息
verb 5
```

## 五、启动 openvpn 服务

```bash
[root@openvpn ~]# systemctl start openvpn@server.service
[root@openvpn ~]# systemctl enable openvpn@server.service

# 无法启动该服务，或者找不到该服务？以下是处理方法
创建文件 vim /lib/systemd/system/openvpn@.service

# openvpn@service的内容
[Unit]
Description=OpenVPN Robust And Highly Flexible Tunneling Application On %I
After=network.target

[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf

[Install]
WantedBy=multi-user.target

```

## 六、配置 iptables 规则（NAT出外网）

为了可以让客户端访问服务器所在局域网的其它主机，需要配置如下规则

```bash
[root@openvpn ~]# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
[root@openvpn ~]# cat /proc/sys/net/ipv4/ip_forward
0
[root@openvpn ~]# echo "1" > /proc/sys/net/ipv4/ip_forward
[root@openvpn ~]# cat /proc/sys/net/ipv4/ip_forward
1
```



## 七、Windows客户端OpenVPN配置

### 7.1 客户端的配置文件（仅供参考，主要通过下面的脚本配置）

```bash
client
dev tun
proto tcp
remote 36.158.226.1 16001
resolv-retry infinite
persist-key
persist-tun
mute-replay-warnings
ca ca.crt
cert yll.crt
key yll.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
comp-lzo
verb 3
mute 20
reneg-sec 0
```

### 7.2 生成客户端证书的脚本如下

```bash
[root@openvpn 3.0.6]# grep -vE "^#|^$" /usr/local/bin/genovpnuser 
if ! rpm -q expect; then
    yum install expect -y
    if ! [ $? == 0 ]; then
        echo "Error: you must install expect manually!"
        exit -1
    fi
fi

dir=/etc/openvpn/3.0.6
if [ $# -ne 1 ]; then
    echo "Usage: $0 USER_NAME"
    echo "-1"
    exit -1
fi
if ! [[ $1 =~ ^[a-zA-Z0-9_]{3,16}$ ]];then
    echo "请使用字母数字或下划线开头的3到16个字符的名字"
    echo "-1"
    exit -1
fi
cd $dir
if ./easyrsa show-cert $1 &> /dev/null;then
    echo "此用户已存在，请重新输入"
    echo "-1"
    exit -1
fi
expect <<-EOF
spawn ./easyrsa gen-req $1 nopass
expect {
"$1" { send "\n" }}
expect eof
EOF
./easyrsa import-req $dir/pki/reqs/${1}.req $1
expect <<-EOF
spawn ./easyrsa sign client $1
expect {
"Confirm" { send "yes\n" }
}
expect eof
EOF

if ! [ -d /client.certs/ ]; then
    mkdir /client.certs/
fi

cat > /client.certs/clientsample.ovpn << EOF
client
dev tun
proto tcp
remote 61.187.64.38 11940
resolv-retry infinite
persist-key
persist-tun
mute-replay-warnings
ca ca.crt
cert sample.crt
key sample.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
comp-lzo
verb 3
mute 20
reneg-sec 0
EOF

if [ -d /client.certs/$1 ]; then
    rm -rf /client.certs/$1
    mkdir /client.certs/$1
else
    mkdir /client.certs/$1
fi
cp $dir/pki/ca.crt /client.certs/$1
cp $dir/pki/ta.key /client.certs/$1
cp $dir/pki/issued/${1}.crt /client.certs/$1
cp $dir/pki/private/${1}.key /client.certs/$1
cd /client.certs
cp clientsample.ovpn client.ovpn
sed -i s@sample.crt@${1}.crt@g client.ovpn
sed -i s@sample.key@${1}.key@g client.ovpn
mv client.ovpn $1/
[root@openvpn 3.0.6]# 
```

### 7.3 吊销客户端证书的脚本如下

```bash
[root@openvpn 3.0.6]# grep -vE "^#|^$" /usr/local/bin/delovpnuser 
if [ $# -ne 1 ]; then
    echo "Usage: $0 USER_NAME"
    echo "-1"
    exit -1
fi
if ! [[ $1 =~ ^[a-zA-Z0-9_]{3,16}$ ]];then
    echo "请使用字母数字或下划线开头的3到16个字符的名字"
    echo "-1"
    exit -1
fi
dir=/etc/openvpn/3.0.6
cd $dir
if ! ./easyrsa show-cert $1 &> /dev/null;then
    echo "没有这个用户,无法删除"
    echo "-1"
    exit -1
fi
expect <<-EOF
spawn ./easyrsa revoke $1
expect {
"revocation" { send "yes\n" }}
expect eof
EOF
./easyrsa gen-crl
rm -rf /client.certs/$1
[root@openvpn 3.0.6]# 
```

### 7.4 创建客户端证书脚本的使用

```shell
# 创建客户端证书
genovpnuser suxianjun

# 将生成如下文件夹
/client.certs/suxianjun

# 将这个文件夹拷贝出来
在wndows的openvpn上，将里面的ovpen文件上传上去，然后点击Connect即可
```







