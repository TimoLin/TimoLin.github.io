---
title: ICEMCFD 18.0/19.0 start problem in UBUNTU 18.04
date: 2018-09-26
permalink: /posts/2018/09/iceminubuntu1804/
categories:
  - CFD
tags:
  - Ubuntu
  - ANSYS
  - ICEM
weight: 1
---
Check my post in [cfd-online](https://www.cfd-online.com/Forums/ansys/203041-installation-ansys-19-ubuntu-18-4-a.html#6)  
At August, I upgraded my UBUNTU 16.04 to 18.04. This version is great.  
However, I met a problem when starting ICEMCFD 18.0 and 19.0.  
I check the start script file of icemcfd at "ansys_inc/v191/icemcfd/linux64_amd/bin". There are two solutions: 

## Solution 1
If you don't want to change the icemcfd start script, just run the following command:  
```shell
icemcfd -log tmp.log
```
-log is a icemcfd argument, tmp.log is the log file generated at your pwd.

## Solution 2
If you use other argument when you start icemcfd, the following solution may be not very good. (I don't use arguments when start icemcfd, so it works well for me)
First- cd to your icemcfd bin dir:
```shell
cd /usr/ansys_inc/v191/icemcfd/linux64_amd/bin
```
Then: edit the icemcfd shell script by any text editor you want, here we use gvim:
```shell
gvim icemcfd
```
change line 607:
from 
`eval "$ICEM_ACN/bin/med" "$args"` 
to
`"$ICEM_ACN/bin/med" $args` or `$ICEM_ACN/bin/med`
