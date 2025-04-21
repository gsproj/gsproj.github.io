---
title: OpenVPN部署手册
date: 2022-07-11 9:29:52
categories:
- 杂记
tags:
---

# OpenVPN部署手册

# 一、安装OpenVPN

```bash
# 红帽系列
yum install easy-rsa openvpn -y
# Ubuntu系
apt-get install easy-rsa openvpn -y
```

# 二、使用easy-rsa生成证书

1、将easy-rsa工具拷贝到openvpn文件夹中

```shell
(base) root@hr650-1:/etc# cd /etc/openvpn/
(base) root@hr650-1:/etc/openvpn# ls
client  server  update-resolv-conf
(base) root@hr650-1:/etc/openvpn# cp /usr/share/easy-rsa/ . -r
(base) root@hr650-1:/etc/openvpn# ls easy-rsa/
easyrsa  openssl-easyrsa.cnf  vars.example  x509-types
```

2、复制变量文件`vars`，并修改参数

```shell
(base) root@hr650-1:/etc/openvpn# cd easy-rsa/
(base) root@hr650-1:/etc/openvpn/easy-rsa# ls
easyrsa  openssl-easyrsa.cnf  vars.example  x509-types
(base) root@hr650-1:/etc/openvpn/easy-rsa# cp vars.example vars

# 参数如下
(base) root@hr650-1:/etc/openvpn/easy-rsa# grep -v "^#\|^$" vars
if [ -z "$EASYRSA_CALLER" ]; then
	echo "You appear to be sourcing an Easy-RSA *vars* file. This is" >&2
	echo "no longer necessary and is disallowed. See the section called" >&2
	echo "*How to use this file* near the top comments for more details." >&2
	return 1
fi
set_var EASYRSA_REQ_COUNTRY	"CN"
set_var EASYRSA_REQ_PROVINCE	"HuNan"
set_var EASYRSA_REQ_CITY	"ChangSha"
set_var EASYRSA_REQ_ORG		"Zst Corporation"
set_var EASYRSA_REQ_EMAIL	"haris@zst.com.cn"
set_var EASYRSA_REQ_OU		"Product Department"
```

3、创建证书

```bash
# 初始化easy-esa
./easyrsa init-pki
# 生成CA证书，生成ca.crt
./easyrsa build-ca nopass（回车）
# 服务端证书，生成server.key
./easyrsa gen-req server nopass (回车)
# 签署服务端证书，生成server.crt
./easyrsa sign server server（输入yes，回车）
# 生成DH证书
./easyrsa gen-dh
# 生成HMAC密钥（防DDos攻击、UDP淹没等恶意攻击）
openvpn --genkey --secret ta.key
```

# 三、配置OpenVPN服务端

1、创建文件`/etc/openvpn/server/server.conf`，内容如下：

```shell
port 2294
# 改成tcp，默认使用udp，如果使用HTTP Proxy，必须使用tcp协议
proto tcp
dev tun
# 证书路径
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
tls-auth /etc/openvpn/easy-rsa/ta.key 0
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

2、启动 openvpn 服务

```bash
# 红帽系列
systemctl enable openvpn@server.service
systemctl start openvpn@server.service
# Ubuntu系列
systemctl enable openvpn-server@server
systemctl start openvpn-server@server
```

3、确保服务正常启用

```shell
(base) root@hr650-1:/etc/openvpn/server# systemctl status openvpn-server@server
● openvpn-server@server.service - OpenVPN service for server
     Loaded: loaded (/usr/lib/systemd/system/openvpn-server@.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-04-03 17:19:31 CST; 1s ago
       Docs: man:openvpn(8)
             https://openvpn.net/community-resources/reference-manual-for-openvpn-2-6/
             https://community.openvpn.net/openvpn/wiki/HOWTO
....
```

# 四、配置 iptables 规则

让客户端可以访问到服务器所在局域网的其它主机

```bash
[root@openvpn ~]# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
[root@openvpn ~]# cat /proc/sys/net/ipv4/ip_forward
0
[root@openvpn ~]# echo "1" > /proc/sys/net/ipv4/ip_forward
[root@openvpn ~]# cat /proc/sys/net/ipv4/ip_forward
1
```



# 五、Windows客户端OpenVPN配置

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
(base) root@hr650-1:/etc/openvpn/easy-rsa# cat /usr/local/bin/genovpnuser 
if ! rpm -q expect; then
    yum install expect -y
    if ! [ $? == 0 ]; then
        echo "Error: you must install expect manually!"
    fi
fi

dir=/etc/openvpn/easy-rsa
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
remote 111.22.183.209 8890
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
cp $dir/ta.key /client.certs/$1
cp $dir/pki/issued/${1}.crt /client.certs/$1
cp $dir/pki/private/${1}.key /client.certs/$1
cd /client.certs
cp clientsample.ovpn client.ovpn
sed -i s@sample.crt@${1}.crt@g client.ovpn
sed -i s@sample.key@${1}.key@g client.ovpn
mv client.ovpn $1/
```

### 7.3 吊销客户端证书的脚本如下

```bash
(base) root@hr650-1:/etc# grep -vE "^#|^$" /usr/local/bin/delovpnuser 
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
dir=/etc/openvpn/easy-rsa
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
```

### 7.4 创建客户端证书脚本的使用

```shell
# 创建客户端证书
genovpnuser haris

# 将生成如下文件夹
/client.certs/haris

# 将这个文件夹拷贝出来
在wndows的openvpn上，将里面的ovpen文件上传上去，然后点击Connect即可
```







