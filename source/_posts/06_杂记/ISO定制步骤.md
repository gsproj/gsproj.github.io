---
title: 母盘ISO定制步骤
date: 2023-05-24 10:10:52
categories:
- 杂记
tags:
---

# 母盘ISO定制步骤

# 一、环境准备

## 1.1 镜像文件和操作系统

ISO修改环境的操作系统和需要定制修改的操作系统ISO镜像：

**Kylin-Desktop-V10-SP1-General-Release-2203-ARM64.iso**

进入操作系统，操作以下命令

```shell
# 切换到root用户
sudo su 

# 安装必要软件
sudo apt-get install squashfs-tools genisoimage

# 切换到/opt目录，创建四个文件夹
# /opt/mnt - 挂载镜像文件
# /opt/livecd - 镜像文件内容拷贝，用于最后打包成新镜像
# /opt/mylivecd - 镜像文件内容拷贝，用于将软件、补丁集成到镜像中，生成filesystem.squashfs等文件
# /opt/tool - 用于存放镜像ISO等文件
cd /opt	&& mkdir mnt livecd tool mylivecd 

# 挂载需要修改的镜像文件，到/opt/mnt
mount -o loop /opt/tool/Kylin-Desktop-V10-SP1-General-Release-2203-ARM64.iso  /opt/mnt/

# 将镜像内的文件拷贝出来
rsync -av /opt/mnt/ /opt/livecd/	
rsync -av /opt/mnt/ /opt/mylivecd/	

# 取消挂载ISO镜像
umount /opt/mnt/
```

# 二、修改镜像文件

先执行以下命令：

```shell
# 切换到mylivecd目录
cd /opt/mylivecd/casper/
# 执行unsquashfs命令
unsquashfs filesystem.squashfs
```

## 2.1 导入补丁和预装软件包

安装1107补丁、PKS补丁及预装软件deb包

1、分别创建三个文件夹，并将需要安装的补丁、软件拷贝到三个文件夹中

```shell
cd /opt/livecd && mkdir 1107 option pks 
# /opt/livecd/1107 - 存放1107补丁
# /opt/livecd/pks - 存放PKS补丁
# /opt/livecd/option - 存放定制软件deb包，如数科OFD、金山WPS等
```

2、修改.kylin-post-actions文件

```shell
vim /opt/livecd/.kylin-post-actions
```

修改后的内容如下：

```shell
#!/bin/bash

rm -rf /etc/apt/apt.conf.d/00recommends
rm -rf /etc/apt/apt.conf.d/00secure
rm -rf /etc/modules-load.d/cups-filters.conf

cat > /etc/apt/sources.list <<EOF
deb http://archive.kylinos.cn/kylin/KYLIN-ALL 10.1 main restricted universe multiverse
deb http://archive.kylinos.cn/kylin/KYLIN-ALL 10.1-2203-updates main restricted universe multiverse
deb http://archive2.kylinos.cn/deb/kylin/production/PART-V10-SP1/custom/partner/V10-SP1 default all
EOF

rm -rf /etc/apt/sources.list.d/*
rm -rf /etc/apt/preferences.d/*

#解决网络安装后不能设置IP bug.
mv /opt/interfaces /etc/network/

mkdir -p /etc/apt/preferences.d/
mkdir -p /etc/apt/sources.list.d/

##解决xrdp远程桌面ssl文件访问权限问题，后续要查清楚原因，用脚本改感觉有安全漏洞
##包列表新增xrdp、xorgxrdp,kylin.chroot脚本增加.xsession
chmod -R a+x /etc/ssl/private
chmod -R a+r /etc/ssl/private
chmod a+r /etc/ssl/private/ssl-cert-snakeoil.key

# 移除社区内核
dpkg -P linux-headers-5.4.0-29 linux-headers-5.4.0-29-generic linux-image-5.4.0-29-generic linux-modules-5.4.0-29-generic linux-modules-extra-5.4.0-29-generic

#安装奇安信杀毒
#dpkg -i /opt/QAXS/*.deb
#rm /opt/QAXS -rf

# sshd 服务异常， 缺少key文件，dpkg-reconfigure openssh-server 能解决问题。
dpkg-reconfigure openssh-server

# 第三方包因安装后过大，后置安装
dpkg -i /opt/third/*.deb
rm /opt/third -rf

# ----制作母盘修改 start----

# 1、补丁集安装，后置安装
bash /tmp/k-i/1108/other.sh hn123123
rm /tmp/k-i/1107 -rf

# 2、pks补丁安装，后置安装
sudo dpkg -i /tmp/k-i/pks/libblkid1_2.34-0.1kylin18_arm64.deb &&sudo dpkg -i /tmp/k-i/pks/1/*.deb
### 1、删除
sudo mv /etc/resolv.conf /etc/resolv.conf.bak
### 2、创建软连接
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
rm /tmp/k-i/pks -rf

# 3、第三方包安装
dpkg -i /tmp/k-i/option/*.deb
rm /tmp/k-i/option -rf

# ----制作母盘修改 end----

# （本次dpkg，放在本脚本的最后执行）
# 安全默认开启包安装20210604，bjca包在安装时安装20210611
dpkg -i /opt/kscset/*.deb
rm /opt/kscset -rf
dpkg -P ksc-set

#关闭sshd服务
systemctl disable sshd

```

## 2.2  导入办公桌面

1、将**办公桌面deb安装包**和**PKS-C1使用指导**拷贝到镜像文件夹

```shell
cp officedesktop_ft2000_kylinv10sp1_cec_0.0.8.91-a20_20230324.run  /opt/mylivecd/casper/squashfs-root/home/

cp PKS-C1项目用户使用指导.pdf  /opt/mylivecd/casper/squashfs-root/home/
```

2、拷贝办公桌面**rc.local**文件到虚拟镜像，用于自动安装办公桌面

```shell
cp rc.local  /opt/mylivecd/casper/squashfs-root/etc/ 
chmod 755 /opt/mylivecd/casper/squashfs-root/etc/rc.local 
```

rc.local文件内容如下

```shell
#!/bin/bash

stat=`getstatus|grep exec|awk -F ' ' '{print $3}'`
if [ "$stat" = "off" ];then
        echo "已经是关闭状态"
else
        setsignstatus off
fi

sudo su - root
#if [ -f ];then
#       dpkg -i
#       sleep 3
#fi

##办公桌面安装
if [ -f /home/officedesktop_ft2000_kylinv10sp1_cec_0.0.8.91-a20_20230324.run ] ;then
        chmod +x /home/officedesktop_ft2000_kylinv10sp1_cec_0.0.8.91-a20_20230324.run
        /home/officedesktop_ft2000_kylinv10sp1_cec_0.0.8.91-a20_20230324.run -c -p all -V
        if [ $? -eq 0 ];then
                rm -f /home/officedesktop_ft2000_kylinv10sp1_cec_0.0.8.91-a20_20230324.run
                cp /home/*.pdf /home/greatwall/桌面/
                rm -f /home/*.pdf
                cd  /usr/share/applications/ && cp qaxbrowser-safe.desktop officedesktop.desktop cn.lanxin.desktop qaxbrowser-safe.desktop wps-office-et.desktop wps-office-official.desktop wps-office-pdf.desktop wps-office-wpp.desktop wps-office-wps.desktop /home/greatwall/桌面/ && chmod 755 /home/greatwall/桌面/*.desktop
                setsignstatus on
        fi
fi


if [ -f /etc/rc.local ];then
        at now+5 minutes <<< "find /etc/ -name rc.local -exec rm -rf {} \;"
fi
```

3、编辑z.sh文件

```shell
vim /opt/mylivecd/casper/squashfs-root/usr/share/kylin-os-installer/scripts/z.sh
```

修改第56行：执行post脚本项

```shell
....
# 执行 post 脚本
do_cdrom_post_actions() {
  if [[ -f /cdrom/.kylin-post-actions ]] && [[ "${is_ghost}" != "true" ]]; then
    cp /cdrom/.kylin-post-actions /target/${IN_TARGET}
    cp -r /cdrom/option /target/${IN_TARGET}	# 新增
    cp -r /cdrom/pks /target/${IN_TARGET}	# 新增
    cp -r /cdrom/1108 /target/${IN_TARGET}	# 新增
    chroot /target bash ${IN_TARGET}/.kylin-post-actions >>${LOG_FILE} 2>&1
  fi

  if [[ -f /cdrom/.kylin-post-actions-nochroot ]] && [[ "${is_ghost}" != "true" ]]; then
    bash /cdrom/.kylin-post-actions-nochroot >>${LOG_FILE} 2>&1
  fi
}
....
```

## 2.3 其他文件拷贝

```shell
# 拷贝pwquality.conf，该文件需事先准备，内容可见附录
cp /opt/tool/pwquality.conf /opt/mylivecd/casper/squashfs-root/etc/security/
# 清空.bash_history，这里拷贝了一个文件过去，文件内容为空
cp /opt/tool/.bash_history /opt/mylivecd/casper/squashfs-root/root/
# 删除filesystem.squashfs文件
cd /opt/mylivecd/casper
rm -rf filesystem.squashfs 
# 打包squashfs-root
mksquashfs squashfs-root filesystem.squashfs -comp xz
# 输出文件系统大小
printf $(du -sx --block-size=1 squashfs-root |cut -f1) >filesystem.size
# 进到虚拟root环境，将已安装的软件列表导出
chroot squashfs-root/ dpkg-query -W >filesystem.manifest
# 执行拷贝文件操作
cp /opt/mylivecd/casper/filesystem.squashfs /opt/livecd/casper/
cp /opt/mylivecd/casper/filesystem.size /opt/livecd/casper/
cp /opt/mylivecd/casper/filesystem.manifest /opt/livecd/casper/
# 切换到livecd目录
cd /opt/livecd/
# 重新生成md5文件
rm -rf md5sum.txt
find -type f -print0 | xargs -0 md5sum|grep -v isolinux/boot.cat |tee md5sum.txt
```

# 三、打包ISO镜像

```shell
# 进到livecd目录
cd /opt/livecd/

# 执行打包命令
mkisofs -J -r -T -joliet-long -V "Kylin-Desktop-V10-SP1" -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -o  /opt/tool/Kylin-Desktop-V10-SP1-CC-PKS-1107.iso /opt/livecd/
```

# 四、附录

1、pwquality.conf文件内容

```shell
# Configuration for systemwide password quality limits
# Defaults:
#
# Number of characters in the new password that must not be present in the
# old password.
# difok = 1
#
# Minimum acceptable size for the new password (plus one if
# credits are not disabled which is the default). (See pam_cracklib manual.)
# Cannot be set to lower value than 6.
minlen = 6
#
# The maximum credit for having digits in the new password. If less than 0
# it is the minimum number of digits in the new password.
dcredit = 0
#
# The maximum credit for having uppercase characters in the new password.
# If less than 0 it is the minimum number of uppercase characters in the new
# password.
ucredit = 0
#
# The maximum credit for having lowercase characters in the new password.
# If less than 0 it is the minimum number of lowercase characters in the new
# password.
lcredit = 0
#
# The maximum credit for having other characters in the new password.
# If less than 0 it is the minimum number of other characters in the new
# password.
ocredit = 0
#
# The minimum number of required classes of characters for the new
# password (digits, uppercase, lowercase, others).
minclass = 1
#
# The maximum number of allowed consecutive same characters in the new password.
# The check is disabled if the value is 0.
maxrepeat = 0
#
# The maximum number of allowed serial characters of the same class in the
# new password.
# The check is disabled if the value is 0.
maxsequence = 0
#
# The maximum number of allowed consecutive characters of the same class in the
# new password.
# The check is disabled if the value is 0.
maxclassrepeat = 0
#
# Whether to check for the words from the passwd entry GECOS string of the user.
# The check is enabled if the value is not 0.
# gecoscheck = 0
#
# Whether to check for the words from the cracklib dictionary.
# The check is enabled if the value is not 0.
dictcheck = 1
#
# Whether to check if it contains the user name in some form.
# The check is enabled if the value is not 0.
usercheck = 1
#
# Whether the check is enforced by the PAM module and possibly other
# applications.
# The new password is rejected if it fails the check and the value is not 0.
# enforcing = 1
#
# Path to the cracklib dictionaries. Default is to use the cracklib default.
dictpath = /var/cache/cracklib/cracklib_dict
#
# Prompt user at most N times before returning with error. The default is 1.
# retry = 1
#
# Enforces pwquality checks on the root user password.
# Enabled if the option is present.
enforce_for_root
#
# Skip testing the password quality for users that are not present in the
# /etc/passwd file.
# Enabled if the option is present.
# local_users_only
#
# Whether to check the new password is a palindromic or not
# Enabled if the option is present
palindrome
#
# Whether to check the new password is simliar with old one
# Check include only case changes and rotated
# Disabled if the option is present
no_similar_check
```



 

 