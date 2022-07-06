---
title: 运维之基础命令--文件ACL
date: 2022-07-06 11:19:52
categories:
- 运维
- （一）基础命令
tags:
---



# 笔记



> 补充：
>
> ​	权限的归属
>
> ​		a : 属组、属主以及其他人的权限一起设置。
>
> ​		o : 其他人
>
> ​		g : 属组
>
> ​		u : 属主



## umask

> 就是解决目录及文件的默认权限。

- 文件的最高权限是多少     777
- 文件夹的最高权限是多少  777



## ACL 

> ACL是为了解决某种特殊环境下的，用户权限需求。

- setfacl ： 设置acl权限
- getfacl ：查看ACL权限

### acl权限归属

- u : 指定用户
- g : 指定组
- o : 修改其他用户权限
- m : 指定mask权限



注：默认情况下，ACL权限跟普通权限保持一致。



## ACL 的流程

- 1、创建文件

  ```bash
  chmod o+x /root
  chmod o+x /root/xiaochen
  cd xiaochen
  
  [root@localhost xiaochen]# touch abc.txt
  [root@localhost xiaochen]# chmod 000 abc.txt 
  [root@localhost xiaochen]# ll
  total 0
  ---------- 1 root root 0 Mar 16 11:39 abc.txt
  ```

- 编写文件

  ```bash
  [root@localhost xiaochen]# echo 111 > abc.txt 
  [root@localhost xiaochen]# cat abc.txt 
  111
  ```

- 设置ACL权限

  ```bash
  [root@localhost xiaochen]# useradd xiaozhang
  [root@localhost xiaochen]# setfacl -m u:xiaozhang:r abc.txt 
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  user:xiaozhang:r--
  group::---
  mask::r--
  other::---
  
  # 注：
  setfacl -m u:用户名称:权限(rwx) 文件名称
  ```

- 查看文件

  ```bash
  [root@localhost ~]# su - xiaozhang
  [xiaozhang@localhost ~]$ cat /root/xiaochen/abc.txt
  111
  ```

  

mask :  rw-

xxxx :   -w-



-w-





## ACL权限的删除

- 删除某个权限

  ```bash
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  user:xiaozhang:r--
  group::---
  group:xiaochen:r-x		#effective:r--
  mask::rw-
  other::r--
  
  [root@localhost xiaochen]# setfacl -x u:xiaozhang abc.txt
  
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  group::---
  group:xiaochen:r-x
  mask::r-x
  other::r--
  
  
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  group::---
  group:xiaochen:r-x
  mask::r-x
  other::r--
  
  [root@localhost xiaochen]# setfacl -x g:xiaochen abc.txt 
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  group::---
  mask::---
  other::r--
  
  ```

  

- 清空acl权限

  ```bash
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  user:xiaochen:rw-
  user:xiaocao:rw-
  group::---
  group:xiaochen:rw-
  group:xiaocao:rw-
  mask::rw-
  other::r--
  
  [root@localhost xiaochen]# setfacl -b abc.txt 
  [root@localhost xiaochen]# getfacl abc.txt 
  # file: abc.txt
  # owner: root
  # group: root
  user::---
  group::---
  other::r--
  
  ```

## ACL继承

> 默认情况下，ACL是不会继承上层目录的权限的。只有目录设置可继承子集文件才可以继承ACL权限。

```bash
[root@localhost linux12]# setfacl -m d:u:xiaochen:w ../linux12
[root@localhost linux12]# touch bcd.txt
[root@localhost linux12]# ls -l
total 0
-rw-r--r--  1 root root 0 Mar 16 15:40 abc.txt
-rw-rw-r--+ 1 root root 0 Mar 16 15:43 bcd.txt
[root@localhost linux12]# getfacl bcd.txt 
# file: bcd.txt
# owner: root
# group: root
user::rw-
user:xiaochen:-w-
group::r-x			#effective:r--
mask::rw-
other::r--

[root@localhost linux12]# 
```













