---
title: 运维之基础命令--sudo和su
date: 2022-07-06 11:19:52
categories:
- 运维
- （一）基础命令
tags:
---



# sudo

> 用于普通用提升权限的。

- 相关的文件：`/etc/sudoers`

- 检查`/etc/sudoers`是否修改正确：visudo -c

- sudoers文件格式

  ```bash
  tom       ALL=           (ALL)          ALL
  用户名称   所有机器可登陆    所有IP或主机名   所有的指令
  ```

- 指令编写格式

  ```bash
  # 必须写全路径：which查看命令全路径
  
  ## 只支持vim命令提权
  xianchen ALL=(ALL)  /usr/bin/vim
  
  ## 支持所有的命令提权
  tom ALL=(ALL)  ALL
  
  ## 不支持某个命令提权
  tom ALL=(ALL) ALL, !/usr/bin/vim
  
  ## 不支持某个命令的部分功能
  xiaochen ALL=(ALL)   ALL, !/usr/bin/vim /root/123.txt
  ```

  



# su

- su - xxx  和 su xxx之间区别

  ```bash
  1、su - xxx ：相当于切换一个窗口，su xxx 仅仅切换了用户
  
  2、su - xxx ： 切换用户执行的系统文件要多于 su xxx
  
  3、su - xxx 是登录
     su  xxx  切换用户
  ```

  

