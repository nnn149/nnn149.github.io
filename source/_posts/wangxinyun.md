---
title: 网心云
comments: true
tags:
  - 网心云
categories:
  - tool
date: 2022-02-13 23:18:12
updated:
---

安装网心云

<!--more-->

[Linux创建虚拟硬盘](https://unixcop.com/how-to-create-a-virtual-hard-disk-in-ubuntu/)

```bash
#查看设备
lsblk
#查看分区
df -lh

dd if=/dev/vda2 of=wxy.img bs=1M count=51200
# if=/dev/zero 初始化数据存储的输入文件
# of=wxy.img 要创建为存储卷的图像文件
# bs=1M 读/写 1mb 速度
# count=1200 51200 Mb 输入块的复制限制(50G)
```



[容器魔方 树莓派安装激活教程](https://help.onethingcloud.com/7cb4/3ed5/8907)

```bash
#自动挂载U盘
#用 fdisk -l 命令查看刚刚接入USB的U盘/硬盘并记下其设备名称
blkid /dev/sda1
#记录uuid
#/dev/sda1: UUID="1c4dff5e-8d78-3c47-aeea-879e949222e9" TYPE="ext4" PARTUUID="08b76b0d-01"
#创建要挂载到的目录
mkdir /root/360u
#打开/etc/fstab文件，在最后行追加(记住是追加，原有的内容不要动)下面内容：
nano /etc/fstab
UUID=1c4dff5e-8d78-3c47-aeea-879e949222e9   /root/360u   ext4    defaults    0 0
#给目录权限
chmod -R 777 /root/360u
#重启，看/root/360u里是否有lost+found目录
```

