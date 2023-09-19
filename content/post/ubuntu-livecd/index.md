---
title: Ubuntu 20.04卡在启动界面的一个解决办法
description: 
date: 2023-09-19 00:00:00+0000
image: cover.png
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---
## 问题描述
Ubuntu 20.04在某次重启后卡在了启动界面，屏幕上有如下的信息：
```
[  0.260118] tpm tpm0: [Firmware Bug]: TPM interrupt not working, polling instead
/dev/nvme0n1p3: clean, 200134/844470 files, 3645600/3777792 block
```
此时即无法进入tty界面（按ctrl+alt+F1至F6），又无法进入grub或者recovery

## 解决方案
有一个解决方法是先进入Ubuntu LiveCD，将硬盘上的系统挂载到LiveCD上，再查找原因。[参考该方法](https://askubuntu.com/a/1107186)，将步骤翻译如下.  
### 从U盘启动
进入Uubunt 20.04 live cd，选择Try Ubuntu
### 查找并挂载硬盘上的Ubuntu系统分区
如果不知道分区是哪个，可以使用GParted查看，或者打开终端输入`sudo fdisk -l`查看。  
```sh
#注意：将nvmeXX或nvmeXY替换为你自己的分区
#挂在根目录分区
sudo mount /dev/nvmeXX /mnt
#如果有对boot做单独分区
sudo mount /dev/nvmeXY /mnt/boot
```
### 将硬盘系统分区挂载到LiveCD系统上
```sh
sudo mount --bind /dev /mnt/dev &&
sudo mount --bind /dev/pts /mnt/dev/pts &&
sudo mount --bind /proc /mnt/proc &&
sudo mount --bind /sys /mnt/sys
```
### 更改root目录
```sh
sudo chroot /mnt
```
### 查找系统bug的原因
例如，本文中是因为libstdc++.so.6和nvidia显卡驱动的原因。  
首先，重新建立libstdc++.so.6的软链接：
```sh
# 进入目录
cd /mnt/usr/lib/x86_64-linux-gnu
# 查看libstdc++.so.6
ls -lht libstdc++.so.6
# 删除原始链接并重新链接到libstdc++.so.6.0.28
sudo rm libstdc++.so.6
sudo ln -s ./libstdc++.so.6.0.28 ./libstdc.so.6
```
卸载nvidia驱动
```sh
sudo apt remove --purge nvidia-*
sudo apt install ubuntu-desktop
echo 'nouveau' | tee -a /etc/modules
sudo apt install ubuntu-desktop
```
如果apt有网络链接问题：
```sh
echo "nameserver 8.8.8.8" >/etc/resolv.conf
```
### 卸载硬盘分区
```sh
exit &&
sudo umount /mnt/sys &&
sudo umount /mnt/proc &&
sudo umount /mnt/dev/pts &&
sudo umount /mnt/dev &&
sudo umount /mnt
```
### 重启系统
