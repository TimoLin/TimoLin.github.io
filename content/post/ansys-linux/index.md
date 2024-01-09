---
title: Linux下Ansys安装与运行的常见问题
description: Ansys under CentOS/Ubuntu
date: 2024-01-09 00:00:00+0000
image: cover.png
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
categories:
  - Linux
tags:
  - CentOS
  - Ubuntu
  - Ansys
---

## 1. 打开Fluent/ICEM出现许可证错误

- 版本：22R1
- 权限：root安装至`/usr/ansys_inc`，普通用户使用
- 系统：CentOS 7

查看对应的日志文件 `~/.ansys/ansyscl.localhost.*.log`：
```log
2024/01/08 23:10:12    INFO                ANSYSLI_CMD=/usr/ansys_inc/v221/licensingclient/linx64/ansyscl -acl 19510.29709 -nodaemon -log ~/.ansys/ansyscl.localhost.19510.29709.log
2024/01/08 23:10:12    INFO                ANSYSLI_INITIALIZATION_FILE='/usr/ansys_inc/shared_files/licensing/ansyslmd.ini'
2024/01/08 23:10:12    INFO                ANSYSLI_PRODORD_FILE='/usr/ansys_inc/shared_files/licensing/prodord/ansysli.prodord.xml'
2024/01/08 23:10:12    INFO                ANSYSLI_TIMEOUT_FLEXLM=15
2024/01/08 23:10:12    ERROR               /usr/ansys_inc/shared_files/licensing/ansyslmd.ini is invalid.
                Could not read option file /usr/ansys_inc/shared_files/licensing/ansyslmd.ini.
2024/01/08 23:10:12    ERROR               Exiting.
```
发现Ansys的许可证管理器无法正确读取配置文件，排除`shared_files`下的文件不存在等弱智原因后，可能和文件夹的权限有关系： 
```sh
# 查看文件权限信息
ls -lht /usr/ansys_inc/shared_files/
```
输出如下，文件的所有权为root，是导致许可证无法读取的原因：
```
drwxr-xr-x. 9 root root 159 May 12  2023 licensing
drwxr-xr-x. 3 root root  20 Nov 13  2021 bin
drwxr-xr-x. 9 root root 112 Nov 13  2021 lib
drwxr-xr-x. 3 root root  20 Nov 13  2021 syslib
```
修改文件夹所有权为普通用户，以解决许可证文件访问问题：
```sh
# 进入root
su
# 修改权限
chown $USER -R /usr/ansys_inc/shared_files
```
打开Fluent或ICEM，查看问题是否已解决。

## 2. 打开ICEM报错
终端输入`icemcfd`，报错如下：
```log
ICEM_ACN is "/usr/ansys_inc/v221/icemcfd/linux64_amd".
LD_LIBRARY_PATH is "/usr/ansys_inc/v221/icemcfd/linux64_amd/lib:/usr/ansys_inc/v221/icemcfd/linux64_amd/bin:/usr/ansys_inc/v221/icemcfd/linux64_amd/dif/iges:/usr/ansys_inc/v221/icemcfd/linux64_amd/../../Framework/bin/Linux64:/usr/ansys_inc/v221/icemcfd/linux64_amd/../../tp/IntelCompiler/2019.3.199/linx64/lib/intel64:/usr/ansys_inc/v221/icemcfd/linux64_amd/../../tp/qt_fw/5.9.6/Linux64/lib:/usr/ansys_inc/v221/icemcfd/linux64_amd/../../tp/hdf5/1_10_5/linx64/lib".
args = 
Window information: depth = 24
                    class = 4
visual depth = 24 class = 4
can't load font screen14, using variable
Signal 11 caught!
segmentation violation - exiting after doing an emergency save
Exiting...
Exit from ICEM CFD
```

解决方案，[安装对应的字体文件]()：
- [CentOS](https://thelinuxcluster.com/2020/09/30/fixing-cant-load-screen14-issues-for-ansys-2020-r1/)
  ```sh
  yum install xorg-x11-fonts-*
  ```
- [Ubuntu](https://www.cfd-online.com/Forums/ansys-meshing/89147-error-when-starting-icem-cfd-ubuntu.html)
  ```sh
  apt-get install xfonts-75dpi xfonts-100dpi
  ```

## 3. Ubuntu打开Fluent报错
通常发生在新系统上第一次安装Ansys后，报错如下：
```log
/software/ansys_inc/V22R1/v221/fluent/bin/fluent: 69: [[: not found
/software/ansys_inc/V22R1/v221/fluent/bin/fluent: 813: [[: not found
/software/ansys_inc/V22R1/v221/fluent/bin/fluent: 836: [[: not found
/software/ansys_inc/V22R1/v221/fluent/bin/fluent: 838: [[: not found
/software/ansys_inc/V22R1/v221/fluent/fluent22.1.0/bin/fluent -r22.1.0
/software/ansys_inc/V22R1/v221/fluent/fluent22.1.0/bin/fluent: 112: [[: not found
/software/ansys_inc/V22R1/v221/fluent/fluent22.1.0/bin/fluent: 744: [[: not found
/software/ansys_inc/V22R1/v221/fluent/fluent22.1.0/bin/fluent: 1108: [[: not found
```
Ubuntu默认使用dash，确实一些shell特性，需要将其修改为bash：
```sh
# 运行下面的命令，并在弹出的窗口中选择No
sudo dpkg-reconfigure dash
```
