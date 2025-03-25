---
title: 05-Docker容器数据持久化
date: 2025-3-20 15:12:52
categories:
- 超哥K8S
- （一）Docker容器基础入门
tags:
---

# 一、什么是容器数据持久化？

当容器删除后，里面的数据也会被一起删除，这样非常容易丢失数据，怎么解决这个问题呢？

可以采用将本地存储空间挂载到容器中的方法，这样即使容器被删除了，本地文件还是存在，以此来实现数据的持久保存。

---

# 二、使用数据持久化

## 2.1 挂载数据卷

> 还是使用在上一篇文章中创建的haris/nginx:v1镜像，

1、创建本地文件夹，并添加index.html文件

```shell
[root@master1 ~/Docker-Demo/03-volume/myvolume]#cat index.html 
<h1>
	this is share html page
</h1>
```

2、使用`docker run -v`选项挂载本地数据卷，用以替换容器中的index.html

```shell
# 运行容器mynginx1
docker run -itd --name "mynginx1" -p 80 -v /root/Docker-Demo/03-volume/myvolume:/usr/share/nginx/html haris/nginx:v1
```

3、测试，可以看到容器运行成功，且index.html替换为本地文件夹中的文件，说明挂载生效

![image-20250320161153636](./../../../img/image-20250320161153636.png)

4、挂载的数据卷是**可以多个容器一起使用的**，再运行一个容器看看

```shell
# 运行容器mynginx2
docker run -itd --name "mynginx2" -p 80 -v /root/Docker-Demo/03-volume/myvolume:/usr/share/nginx/html haris/nginx:v1
```

5、测试两个都可以正常访问

![image-20250320161407695](./../../../img/image-20250320161407695.png)

---

## 2.2 给数据卷添加权限

>既然用于挂载的数据卷每个容器都能访问，岂不是很危险，得想方法加个权限，比如只读？

1、在挂载时，使用`ro`标签给数据卷添加只读权限

```shell
docker run -itd --name "mynginx-ro" -v /root/Docker-Demo/03-volume/myvolume:/usr/share/nginx/html:ro -p 80 haris/nginx:v1
```

2、进入容器，在里面写数据是不允许的

```shell
# 进入容器内部
[root@master1 ~/Docker-Demo/03-volume/myvolume]#docker exec -it mynginx-ro /bin/bash         
[root@e7c522293e5d /]# cd /usr/share/nginx/html/
[root@e7c522293e5d html]# ls
index.html
# 文件夹可读
[root@e7c522293e5d html]# cat index.html 
<h1>
	this is share html page
</h1>
# 但是不能写，只读文件系统
[root@e7c522293e5d html]# echo "destory you" > index.html 
bash: index.html: Read-only file system
```

