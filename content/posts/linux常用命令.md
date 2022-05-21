---
title: linux常用命令命令
date: 2019-02-13 14:50:27
tag:
---

# 制作linux usb启动盘

#### 格式化u盘

    mkfs.vfat -F 32 -n USB /dev/sdb1

#### 安装syslinux,mtools

    apt-get install syslinux mtools

#### 安装linux引导

    /usr/bin/syslinux /dev/sdb1

#### 写入mbr

    cat /usr/lib/syslinux/mbr/mbr.bin > /dev/sdb

#### 配置重命名

    mv ioslinux.cfg syslinux.cfg

#### 安装镜像

//TODO

#### chroot时绑定

    mount --bind /proc /mnt/proc
    mount --bind /run /mnt/run
    mount --bind /dev /mnt/dev
    mount --bind /sys /mnt/sys

#### 获取文件魔数(判断文件类型)

    xxd file.png | head -n 1 | awk '{print $2$3}'
