---
title: 长城TD120A2安装OpenEuler
date: 2022-07-11 9:29:52
categories:
- 杂记
tags:
---

>记录安装过程

### 一、环境

|          | 型号            |
| -------- | --------------- |
| CPU      | 飞腾D2000/8核   |
| 内存     | 紫光DDR4 3200   |
| 显卡     | AMD RX520       |
| 安装系统 | OpenEuler 20.09 |

### 二、OpenEuler安装

1、镜像准备：

```she
openEuler-20.09-aarch64-dvd.iso
```

2、制作启动盘

```shell
使用win32diskimager制作U盘启动器
```

3、安装

```shell
进入安装界面：
Install openEuler 20.09...
Test this media & install openEuler 20.09...
Troubleshooting

选择到Install openEuler 20.09...行
Ctrl + e 进入编辑界面
删除"video=efifb:off"
Ctrl + x 保存并退出，进入图形化安装界面
```

>为什么要删除此grub参数？ ---直接安装将卡在此界面
>
>EFI stub: Booting Linux Kernel...
>
>EFI stub: EFI_RNG_PROTOCOL unavaliable, no randomness supplied
>
>EFI stub: Using DTB from configuration table
>
>EFT stub: Exiting boot services and installing virtual address map...

4、启动系统

```shell
进入系统后，修改/boot/efi/EFI/openEuler/grub.cfg文件
同样，将"video=efifb:off"注释或者删除
>>防止重启卡在引导界面
```

5、无线网卡连接wifi

>TD120A2安装OpenEuler后,有线网卡无法识别，需要安装USB无线网卡，并安装配套的驱动，无线网卡识别后可以连接wifi

```shell
nmcli dev wifi # 查询wifi
nmcli dev wifi connet 网络名 password 密码
```

6、图形化界面安装

```shell
sudo dnf makecache
sudo dnf groupinstall ukui
systemctl set-default graphical
```