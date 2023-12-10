---
title: CentOS下Miniconda导致VNC无法启动的解决方案
description: Miniconda环境与VNC的冲突解决
date: 2023-10-25 00:00:00+0000
image: cover.png
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
categories:
  - Linux
tags:
  - CentOS
  - Miniconda
  - VNC
---
参考博文[CentOS7.2 部署VNC服务记录](https://www.cnblogs.com/kevingrace/p/5821450.html)。

## 问题描述
首先在`CentOS-7.9`中安装了`tigervnc`，并配置好了`vncserver`的自启动服务： 
```service
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=simple

# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
ExecStart=/usr/sbin/runuser -l userTest -c "/usr/bin/vncserver %i -geometry 1440x900"
ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

[Install]
WantedBy=multi-user.target
```
在未安装Miniconda环境之前，是可以正常启动的，包括重启系统后自启动，或者使用`systemctl start/restart vncserver@\:1.service`进行手动启动/重启。

安装Miniconda后，在普通用户`userTest`的`.bashrc`中会自动添加如下代码自动激活conda环境：
```
__conda_setup="$('/home/nuaa/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/nuaa/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/home/nuaa/miniconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/nuaa/miniconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

然后，`vncserver`服务就无法开机自启动或者使用`systemctl`启动了，具体报错如下：
```
(imsettings-check:31898): GLib-GIO-CRITICAL **: 21:56:03.842: g_dbus_proxy_call_sync_internal: assertion 'G_IS_DBUS_PROXY (proxy)' failed
GLib-GIO-Message: 21:56:03.854: Using the 'memory' GSettings backend.  Your settings will not be saved or shared with other applications.
 
** (process:31798): WARNING **: 21:56:03.861: Could not make bus activated clients aware of XDG_CURRENT_DESKTOP=GNOME environment variable:
Could not connect: Connection refused
```

## 原因
Miniconda环境下的`dbus-daemon`与CentOS系统中的`dubs-daemon`存在冲突：
```
$ which dbus-daemon
~/miniconda3/bin/dbus-daemon
```

## 解决方案
避免普通用户`userTest`下Miniconda环境的自动加载，将自动加载环境的代码修改为下面的bash函数形式，需要使用miniconda环境时，运行`miniconda`来激活环境：
```
miniconda(){
    __conda_setup="$('/home/nuaa/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "/home/nuaa/miniconda3/etc/profile.d/conda.sh" ]; then
            . "/home/nuaa/miniconda3/etc/profile.d/conda.sh"
        else
            export PATH="/home/nuaa/miniconda3/bin:$PATH"
        fi
    fi
    unset __conda_setup
    # <<< conda initialize <<<
}
```
重启vncserver服务或者重启系统：
```sh
systemctl stop vncserver@\:1.service
systemctl start vncserver@\:1.service
```

## VNC无法登录的一个解决方法
有时在CentoOS登录界面输入密码无效，或者无法登录，可能是VNC桌面卡死了。

- 方法1: 重启VNC 
  SSH登录到目标机器，重启VNC的系统服务。但该方法会使得VNC桌面中的任务随之杀掉。
  ```sh
  systemctl stop vncserver@\:1.service
  systemctl start vncserver@\:1.service
  ```
- 方法2： loginctl解锁桌面（参考[博文](https://zhuanlan.zhihu.com/p/507878402)） 
  首先查看当前登录了
  ```sh
  loginctl list-sessions
  ```
  输出如下：
  ```
  SESSION        UID USER             SEAT
        c1       1000 YOUR_USER_NAME
        c2         42 gdm              seat0
     20210       1000 YOUR_USER_NAME

  3 sessions listed.
  ```
  解锁VNC登录界面：
  ```sh
  # 一般VNC的session id可能是数字
  loginctl unlock-session 20210
  ```
