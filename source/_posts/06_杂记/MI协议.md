---
title: MI协议
date: 2022-07-06 11:19:52
categories:
- 杂记
tags:
-
---

# Mi协议使用

## 1 使用mi启动调试

```shell
gs@ft-svr:~/matrix$ gdb --interpreter mi test
=thread-group-added,id="i1"
~"GNU gdb (Ubuntu 8.2.91.20190405-0ubuntu3) 8.2.91.20190405-git\n"
~"Copyright (C) 2019 Free Software Foundation, Inc.\n"
~"License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>\nThis is free software: you are free to change and redistribute it.\nThere is NO WARRANTY, to the extent permitted by law."
~"\nType \"show copying\" and \"show warranty\" for details.\n"
~"This GDB was configured as \"aarch64-linux-gnu\".\n"
~"Type \"show configuration\" for configuration details.\n"
~"For bug reporting instructions, please see:\n"
~"<http://www.gnu.org/software/gdb/bugs/>.\n"
~"Find the GDB manual and other documentation resources online at:\n    <http://www.gnu.org/software/gdb/documentation/>."
~"\n\n"
~"For help, type \"help\".\n"
~"Type \"apropos word\" to search for commands related to \"word\"...\n"
~"Reading symbols from test...\n"
(gdb)
```

## 2 断点命令（BreakPoint）

### 2.1 -break-after

```shell
-break-after number count
第number个断点在执行count次后有效
```

### 2.2 -break-condition

```shell
-break-condition number expr
第number个断点在表达式expr为true时有效
```

### 2.3 -break-delete

```shell
-break-delete(breakpoint number)+
删除指定number的多个断点
```

### 2.4 -break-disable

```shell
-break-disable(breakpoint number)+
使用指定Number的多个断点失效
```

### 2.5 -break-enable

```shell
-break-enable(breakpoint number)+
使用指定Number的多个断点生效
```

### 2.6 -break-info

```shell
-break-info breakpoint
得到指定断点的信息
```

### 2.7 -break-insert

```shell
-break-insert
	-t				插入临时断点
	-h				插入硬件断点
	-r				插入正则断点，当函数名匹配正则表达式时生效
	-c condition    插入条件断点
	-i ignore-count 插入一个指定无效次数的断点
	-p thread  		
	line | addr(func) 
```

### 2.8. -break-list

```shell
-break-list
显示已插入的断点列表
```

### 2.9 -break-watch

```shell
-break-watch [-a|-r] variable
创建一个观察点，-a标识对variable读写时有效，-r标识只读时有效
```

# 