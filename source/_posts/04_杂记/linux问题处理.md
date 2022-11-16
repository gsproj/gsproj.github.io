1、机器卡顿、慢

测试硬盘读写：

```shell
# 测试写
time dd if=/dev/zero of=/tmp/test bs=8k count=1000000
# 测试读
time dd if=/tmp/test of=/dev/null bs=8k
# 测试读写
time dd if=/tmp/test of=/var/test bs=64k
```

硬盘坏道测试：

```shell
badblocks -s -o -v test.log /dev/sda
```

2、查看NVME硬盘使用时间，机器使用时间

```shell
nvme smart-log /dev/nvme0n1
# 看power_on_hour 
```

查看系统安装时间、文件系统创建时间、挂在次数（开机次数）

```shell
tune2fs -l /dev/sda2

# Filesystem created: 文件系统创建时间即系统安装时间
# Mount count：挂在次数，即开机次数
```

