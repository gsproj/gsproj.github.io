---
title: GCC编译方法
date: 2022-07-06 11:19:52
categories:
- 测试
- 信创POC测试
tags:
-
---

# GCC编译方法

## 一、GCC编译

### 1.1 依赖软件编译

建议将依赖库和GCC安装到相同目录，方便设置环境变量，本文安装路径为`usr/local/gcc1130`

**编译安装gmp**

```shell
./configure --prefix=/usr/local/gcc1130
make 
make install
```

**编译安装mpfr**

```shell
./configure --prefix=/usr/local/gcc1130 --with-gmp=/usr/local/gcc1130
make 
make install
```

**编译安装mpc**

```shell
./configure --prefix=/usr/local/gcc1130 --with-gmp=/usr/local/gcc1130 --with-mpfr=/usr/local/gcc1130
make
make install
```

**编译安装isl**

```shell
./configure --prefix=/usr/local/gcc1130 --with-gmp-prefix=/usr/local/gcc1130
make
make install
```

### 1.2 编译GCC

cofigure参考鲲鹏GCC得到

```shell
./configure --prefix=/usr/local/gcc1130 --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,fortran,lto --enable-plugin --enable-initfini-array --disable-libgcj --without-isl --without-cloog --enable-gnu-indirect-function --build=aarch64-linux-gnu --with-stage1-ldflags=' -Wl,-z,relro,-z,now' --with-boot-ldflags=' -Wl,-z,relro,-z,now' --disable-bootstrap --with-multilib-list=lp64 --enable-bolt --with-gmp=/usr/local/gcc1130  --with-mpfr=/usr/local/gcc1130  --with-mpc=/usr/local/gcc1130  --with-isl=/usr/local/gcc1130 
```

设置依赖库的路径

```shell
export LD_LIBRARY_PATH=/usr/local/gcc1130/lib:$LD_LIBRARY_PATH
```

编译

```shell
make -j 8
make install
```

### 1.3 编译问题

报错信息：

```shell
LIBRARY_PATH variable... contains current directory
```

解决方法：

改问题由LIBRARY_PATH环境变量乱了导致，清空环境变量，重新配置即可

### 1.4 使用GCC

创建gcc.env文件

```shell
#!/bin/bash

export GCC_HOME=/usr/local/gcc1130

export PATH=$GCC_HOME/bin/:$PATH
export LD_LIBRARY_PATH=$GCC_HOME/lib64:$GCC_HOME/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$GCC_HOME/lib64:$GCC_HOME/lib:$LIBRARY_PATH
export C_INCLUDE_PATH=$GCC_HOME/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=$GCC_HOME/include:$CPLUS_INCLUDE_PATH
export CPATH=$GCC_HOME/include:$CPATH
```

使其生效

```shell
source gcc.env
```

测试是否成功

```shell
gcc -v
```

## 二、附：MPICH3编译方法

设置gcc830的环境变量

```shell
export PATH=/home/testcpu/test652/tools/gcc830/bin:$PATH
export LD_LIBRARY_PATH=/home/testcpu/test652/tools/gcc830/lib:$LD_LIBRARY_PATH
export LD_INCLUDE_PATH=/home/testcpu/test652/tools/gcc830/include:$LD_INCLUDE_PATH
export CPATH=/home/testcpu/test652/tools/gcc830/include:$CPATH
export LD_LIBRARY_PATH=/home/testcpu/test652/tools/gcc830/lib64:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/home/testcpu/test652/tools/gcc830/libexec:$LD_LIBRARY_PATH
```

再编译

```shell
./configure --prefix=/home/testcpu/test652/tools/mpi3-gcc830 --enable-fast --enable-shared=yes --enable-threads=runtime --with-ch3-rank-bits=32 --enable-romio --with-file-system=ufs+nfs --with-mpe
make 
make install
```

设置环境变量使用

```shell
export PATH=/home/testcpu/test652/tools/mpi3-gcc830/bin:$PATH
export LD_LIBRARY_PATH=/home/testcpu/test652/tools/mpi3-gcc830/lib:$LD_LIBRARY_PATH
export LD_INCLUDE_PATH=/home/testcpu/test652/tools/mpi3-gcc830/include:$LD_INCLUDE_PATH
```

## 三、附：Redhat设置本地镜像源

挂载镜像

```shell
mount -o loop -t iso9660 /home/IOSYS/test652/images/rhel-server-7.6-x86_64-dvd.iso /home/IOSYS/test652/mnt/CDROM
```

设置本地镜像源

```shell
vim /etc/yum.repos.d/redhat.repo

[rhel7.6]
name=rhel7.6
baseurl=file:///home/testcpu/test652/mnt/CDROM
enabled=1
gpgcheck=0
priority=1
```

使其生效

```shell
yum clean
yum update
yum repolist
```

