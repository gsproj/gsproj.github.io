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

需要依次编译依赖软件，然后在编译gcc中使用

```shell
# 设置isl的库路径
export LD_LIBRARY_PATH=/home/testcpu/test652/tools/gcc830/lib:$LD_LIBRARY_PATH
# 编译GCC
../configure --prefix=/home/testcpu/test652/tools/gcc830 --enable-lto --disable-bootstrap --enable-languages=c,c++,fortran --with-gmp=/home/testcpu/test652/tools/gcc830 --with-mpfr=/home/testcpu/test652/tools/gcc830 --with-mpc=/home/testcpu/test652/tools/gcc830 --with-isl=/home/testcpu/test652/tools/gcc830
```

### 1.1 依赖软件编译

#### 1.1.2 编译安装gmp

```shell
./configure --prefix=/home/testcpu/test652/tools/gcc830
make 
make install

./configure --with-gmp=/home/testcpu/test652/tools/gcc83 --with-mpfr=/home/testcpu/test652/tools/gcc83  --prefix=/home/testcpu/test652/tools/gcc83
```

#### 1.1.3 编译安装mpfr

```shell
./configure --prefix=/home/testcpu/test652/tools/gcc830 --with-gmp=/home/testcpu/test652/tools/gcc830
make 
make install
```

#### 1.1.4 编译安装mpc

```shell
./configure --prefix=/home/testcpu/test652/tools/gcc830 --with-gmp=/home/testcpu/test652/tools/gcc830 --with-mpfr=/home/testcpu/test652/tools/gcc830
make
make install
```

#### 1.1.5 编译安装isl

```shell
./configure --prefix=/home/testcpu/test652/tools/gcc830 --with-gmp-prefix=/home/testcpu/test652/tools/gcc830
```

```shell
 yum install gcc-* gtk2 gtk3 x11 libX11.x86_64 libX11-devel.x86_64 libXorg libXss libXScrnSaver.x86_64 xorg-x11-server-Xorg.x86_64 xulrunner.x86_64 xulrunner.i686 libstdc++.i686 libstdc++-devel.i686
glibc.i686 glibc-devel.i686 libgcc* xulrunner.x86_64 xulrunner.i686 glibc-devel.x86_64 glibc.x86_64 autoconf-archive.noarch gtk2 gtk3 pango libXScrnSaver libX11.x86_64 libX11-devel.x86_64 libX11-common.noarch libxkbcommon-x11.x86_64 xorg-x11-server-common.x86_64 libstdc++* -y
```

```shell
#!/bin/bash
mount -o loop -t iso9660 /home/testcpu/test652/images/rhel-server-7.6-x86_64-dvd.iso /home/testcpu/test652/mnt/CDROM


yum install gcc-* gtk2 gtk3 glibc-devel.i686 x11 libX11.x86_64 libX11-devel.x86_64 libXorg libXss libXScrnSaver.x86_64 xorg-x11-server-Xorg.x86_64 xulrunner.x86_64 xulrunner.i686 libstdc++.i686 libstdc++-devel.i686 glibc* libgcc* xulrunner* autoconf-archive.noarch gtk2 gtk3 pango libXScrnSaver libX11.x86_64 libX11-devel.x86_64 libX11-common.noarch libxkbcommon-x11.x86_64 xorg-x11-server-common.x86_64 libstdc++* -y

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

yum clean
yum update
yum repolist
```

