---
title: Ubuntu-22.04在Docker中编译OpenFOAM-2.3.1
description: Docker+OpenFOAM-2.3.1
date: 2023-09-26 00:00:00+0000
image: cover.png
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

宿主机：Ubuntu-22.04
Docker容器：Ubuntu-18.04
OpenFOAM版本：v2.3.1
## 问题描述
Ubuntu-22.04中的gcc-11版本过高，无法编译低版本OpenFOAM（v2.3.1需要gcc-4.8~5.5）。
## 解决方案
- 创建基于`Ubuntu-18.04`的Docker容器
- 搭建OpenFOAM编译环境
- 建立宿主机`OpenFOAM`代码与算例的目录到docker目录的映射
- 宿主机下修改代码
- Docker容器中编译代码，执行算例
- 宿主机下`Tecplot`/`Paraview`查看结果
## 依赖
1. 查看[官方教程](https://docs.docker.com/engine/install/ubuntu/)安装docker。  
2. 设置[免ROOT使用docker命令](https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo)

## 具体步骤
### 拉取[OpenFOAM231-docker](https://github.com/TimoLin/openfoam231-docker)脚本
例如放到`$HOME/github`文件夹
```sh
cd $HOME/github
git clone https://github.com/TimoLin/openfoam231-docker.git
```
或者
```sh
git clone git@github.com:TimoLin/openfoam231-docker.git
```
### 创建`of231`镜像
运行脚本`prepareOpenFOAM231.sh`:
```sh
./prepareOpenFOAM231.sh
```
输出如下：
```log
*******************************************************
Following Docker containers are present on your system:
*******************************************************
CONTAINER ID   IMAGE          COMMAND                   CREATED       STATUS                   PORTS     NAMES
64ee332b50a6   caecf4ca1e48   "/bin/sh -c 'sh -c \"…"   2 hours ago   Exited (1) 2 hours ago             crazy_benz
WM_CC
*******************************************************

Creating Docker OpenFOAM container openfoam231
8ea1e4429ac9ea43df1c58ed206427383771c9724e757b369686d9440e2dc465
Container openfoam231 was created.
*******************************************************
Run the ./startOpenFOAM script to launch container
*******************************************************
```
## 使用镜像
首先在宿主机`$HOME/OpenFOAM`文件夹下创建OpenFOAM环境变量文件`of-env.bashrc`，内容如下：
```sh
source $HOME/OpenFOAM/OpenFOAM-2.3.1/etc/bashrc WM_NCOMPPROCS=4 WM_MPLIB=SYSTEMOPENMPI
export WM_CC='gcc-5'
export WM_CXX='g++-5'
export WM_NCOMPPROCS=4
export WM_MPLIB=SYSTEMOPENMPI
```

运行脚本`startOpenFOAM231.sh`
```sh
./startOpenFOAM231.sh
```

输出：
```sh
Entering container openfoam231:8ea1e4429ac9...
8ea1e4429ac9% 
```

在docker中激活openfoam环境：
```sh
of231
```
最后可以编译或运行算例了。

## Appendix
创建docker镜像：
```sh
docker build - < Dockerfile
```
打tag：
```sh
docker tag REPLACE_WITH_DOCKER_IMAGE_ID ztnuaa/openfoam231:latest
```
登录dockerhub：
```sh
docker login
```
推送到dockerhub：
```sh
docker push ztnuaa/openfoam231:latest
```

