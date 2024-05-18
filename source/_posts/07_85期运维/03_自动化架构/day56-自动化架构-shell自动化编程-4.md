---
title: day56-自动化架构-shell自动化编程（四）
date: 2024-5-18 15:23:52
categories:
- 运维
- （二）综合架构
tags: 
---

# 自动化架构-shell自动化编程（三）

今日内容：

# 一、脚本中常用的检查命令

## 1.1 概述

| 常用检查命令 | 作用                                   | 示例               |
| ------------ | -------------------------------------- | ------------------ |
| 端口         | 一般用于检查端口是否存在，能否正常连接 | ss / netstat       |
| 进程         | 检查进程状态、指标                     | ps / top / iotop   |
| 网络         | 检查网络连通性                         | ping / iftop / dig |
| web          | http请求                               | curl / wget        |
| 系统全能     | atop(all)                              |                    |

## 1.2 端口检查

### 1.2.1 是否存在

```shell
ss -lntup |grep 80
netstat -lntup |grep 80
lsof -i:80
```



### 1.2.2 能否访问

```shell
# telnet, -e指定逃脱字符，遇到这个字符相当于按ctrl+c
echo q | telnet -eq 10.0.0.61 80
Telnet escape character is 'q'.
Trying 10.0.0.61...
telnet: connect to address 10.0.0.61: Connection
refused

echo $?
1


# nc, -z 无io模式，用于检查端口是否连通
nc -z 10.0.0.61 22

echo $?
0
```



## 1.3 进程检查

ps 、top



## 1.4 网络检查

ping
iftop  



## 1.5 web与api的测试命令

### 1.5.1 curl

curl: 

- -v -L跟随跳转 
- -H 修改请求头 
- -I 只显示响应头
-  -w 按照指定格式输出 
- -o 输出到指定位置的文件
- -s 安静模式，一般使用管道需要加上，可以关闭下载进度显示

案例：

```shell
# curl获取http状态码
curl -s -w '%{http_code}\n' -o /dev/null www.baidu.com
200 

# 获取响应头
[root@mn01 ~]# curl -I www.baidu.com
HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: private, no-cache, no-store, proxyrevalidate, no-transform
```

curl 的 POST 选项

- 输入用户名密码(ak id和secret) 获得令牌 token
- 通过令牌访问资源
- -X 请求方法
- -H 修改请求头
- -d 请求报文主体  

案例：

```shell
curl -s -X POST -H ContentType:application/json-rpc 10.0.0.71/api_jsonrpc.php -d '{
  "jsonrpc": "2.0",
  "method": "user.login",
  "params": {
    "user": "Admin",
    "password": "zabbix"
  },
  "id": 1,
  "auth": null
}'
```

### 1.5.2 wget

wget：

- -t 失败后，重复尝试次数、
- -T timeout 超时时间
- -q 不显示wget输出
- --spider 不下载文件,仅访问.  

## 1.6 全能信息

atop的基本使用

```shell
# 安装
yum install atop -y

# 启动服务
systemctl start atop
systemctl enable atop

# 查看
atop
```

效果：

![image-20240518191240137](../../../img/image-20240518191240137.png)

