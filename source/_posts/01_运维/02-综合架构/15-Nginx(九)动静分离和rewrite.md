---
title: 运维之综合架构--07--Nginx(九)动静分离
date: 2022-07-06 11:19:52
categories:
- 运维
- （二）综合架构
tags:
---



## 一、动静分离介绍

动静分离，通过中间件将动静分离和静态请求进⾏分离；
通过中间件将动态请求和静态请求分离，可以建上不必要的请求消耗，同时能减少请求的延时。
通过中间件将动态请求和静态请求分离，逻辑图如下:  

## 二、单台服务器动静分离配置

逻辑图如下：

![单台服务器动静分离](13-Nginx(九)动静分离和rewrite.assets/单台服务器动静分离.png)

编辑Nginx配置文件

```shell
[root@web01 conf.d]vim blog.conf
server {
    listen 80;
    server_name blog.linux.com;
    location / {
    root /code/wordpress;
    index index.php;
}

# 如果请求的是以.jpg结尾的静态⽂件 就去/code/images⽬录下访问
location ~* \.jpg$ {
    root /code/images;
    }
    location ~* \.php$ {
    root /code/wordpress;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    }
} 

# 创建⽬录
[root@web01 conf.d]# mkdir /code/images/

# 实现动静分离
⽅式⼀：把⽂件挪到/code/images/
cp -r /code/wordpress/wp-content /code/images/

⽅式⼆：做软连接
cd /code
ln -s wordpress images
```

## 三、多台服务器动静分离配置

参考：https://www.cnblogs.com/backups/p/nginx10.html

### 3.1 原理图

![多台服务器动静分离](13-Nginx(九)动静分离和rewrite.assets/多台服务器动静分离.png)

### 3.2 实验环境准备

| 主机  | 外网     | 内网       | 作用服务               |
| ----- | -------- | ---------- | ---------------------- |
| lb01  | 10.0.0.5 | 172.16.1.5 | 负载均衡 nginx proxy   |
| web01 | 10.0.0.7 | 172.16.1.7 | 静态资源 nginx static  |
| web02 | 10.0.0.8 | 172.16.1.8 | 动态资源 tomcat server |

### 3.3 配置web01提供静态资源

1、配置Nginx

```shell
[root@web01 images]# cat /etc/nginx/conf.d/jt.conf
# 动静分离，web01提供静态文件服务
server {
        listen 80;
        server_name dongjing.gs.com;
        
        # 提供临时测试的域名
        root /code/dongjing;
        index index.html;

        location ~* ^.*\.(jpg|png|gif)$ {
                root /code/dongjing/images;
        }
}
```

2、上传静态资源，测试访问静态页面

```shell
[root@web01 ~]# echo "web01...jingtai" > /code/index.html
[root@web01 ~]# mkdir /code/dongjing/images/ && cd /code/dongjing/images/
[root@web01 images]# rz 1.jpg
[root@web01 images]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@web01 images]# systemctl reload nginx
```

`在客户端配置hosts，10.0.0.7 dongjing.gs.com`

打开浏览器访问http://dongjing.gs.com

![image-20210825141423629](13-Nginx(九)动静分离和rewrite.assets/image-20210825141423629.png)

打开浏览器访问http://dongjing.gs.com/1.png

![image-20210825142256349](13-Nginx(九)动静分离和rewrite.assets/image-20210825142256349.png)

### 3.4 配置web02提供动态资源（tomcat + java模拟)

1、安装Tomcat并添加jsp文件

```shell
[root@web02 ~]# yum install -y tomcat
[root@web02 ~]# mkdir /usr/share/tomcat/webapps/ROOT
[root@web02 ~]# cat >/usr/share/tomcat/webapps/ROOT/java_test.jsp <<EOF
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<HTML>
    <HEAD>
        <TITLE>oldboy JSP Page</TITLE>
    </HEAD>
    <BODY>
        <%
            Random rand = new Random();
            out.println("<h1>随机数:<h1>");
            out.println(rand.nextInt(99)+100);
        %>
    </BODY>
</HTML>
EOF
[root@web02 ~]# systemctl start tomcat
```

2、测试访问

访问http://10.0.0.8:8080/java_test.jsp

![image-20210825162829977](13-Nginx(九)动静分离和rewrite.assets/image-20210825162829977.png)

### 3.4 增加负载均衡，实现动静分离

1、配置Nginx

```shell
[root@lb01 dongtai]# cat /etc/nginx/conf.d/proxy_dj.conf
upstream jt {
        server 172.16.1.7:80;
}

upstream dt {
        server 172.16.1.8:8080;
}

server {
        listen 80;
        server_name dongjing.gs.com;

        location ~* ^.*\.(jpg|png|gif)$ {
                proxy_pass http://jt;
                proxy_set_header HOST $http_host;
        }

        location ~ \.jsp$ {
                proxy_pass http://dt;
                proxy_set_header HOST $http_host;
        }
}
```

2、测试访问

`客户端配置hosts，10.0.0.3 dongjing.gs.com`

在浏览器访问

http://dongjing.gs.com/1.png

http://10.0.0.8:8080/java_test.jsp

的效果是一样的，但是访问http://dongjing.gs.com显示的是Nginx默认的主页，因为lb01中暂时没有对应文件夹的index.html

### 3.5 在负载均衡上创建同时调用动态和静态资源的index.html

1、修改Nginx配置

```shell
[root@lb01 dongtai]# cat /etc/nginx/conf.d/proxy_dj.conf
upstream jt {
        server 172.16.1.7:80;
}

upstream dt {
        server 172.16.1.8:8080;
}

server {
        listen 80;
        server_name dongjing.gs.com;
		
		# 新增
        location / {
                root /code/dongjing;
                index index.html;
        }

        location ~* ^.*\.(jpg|png|gif)$ {
                proxy_pass http://jt;
                proxy_set_header HOST $http_host;
        }

        location ~ \.jsp$ {
                proxy_pass http://dt;
                proxy_set_header HOST $http_host;
        }
}
```

2、创建对应的目录和页面

```shell
[root@lb01 /]# mkdir /code/dongjing
[root@lb01 /]# cat /code/dongjing/index.html
<html lang="en">
<head>
        <meta charset="UTF-8" />
        <title>测试ajax和跨域访问</title>
        <script src="http://libs.baidu.com/jquery/2.1.4/jquery.min.js"></script>
</head>
<script type="text/javascript">
$(document).ready(function(){
        $.ajax({
        type: "GET",
        url: "http://dongjing.gs.com/java_test.jsp",
        success: function(data){
                $("#get_data").html(data)
        },
        error: function() {
                alert("哎呦喂,失败了,回去检查你服务去~");
        }
        });
});
</script>
<body>
        <h1>测试动静分离</h1>
        <div id="get_data"></div>
        <img src="http://dongjing.gs.com/1.png">
</body>
</html>
```

3、测试访问

浏览器访问：http://dongjing.gs.com

![image-20210825143746986](13-Nginx(九)动静分离和rewrite.assets/image-20210825143746986.png)

正常负载均衡的现象：

```shell
关掉web02的Tomcat服务，web01服务正常

​	http://dongjing.gs.com可以正常打开，图片正常显示，随机数将显示异常

web02的Tomcat服务正常，关掉web01的nginx服务

​	http://dongjing.gs.com可以正常打开，图片显示异常，随机正常显示
```

## 四、综合案例-Nginx资源分离

### 4.1 什么是资源分离？

```shell
根据浏览器Agent标识可以访问到不同的页面资源：
比如：
    Android访问到的是Android的页面
    PC访问到的是PC的页面
    iphone访问放到的是iphone的页面
```

### 4.2 实验环境准备

| 主机  | 外网     | 内网       | 作用服务             |
| ----- | -------- | ---------- | -------------------- |
| lb01  | 10.0.0.5 | 172.16.1.5 | 负载均衡 nginx proxy |
| web01 | 10.0.0.7 | 172.16.1.7 | 提供Android手机页面  |
| web02 | 10.0.0.8 | 172.16.1.8 | 提供PC访问页面       |

