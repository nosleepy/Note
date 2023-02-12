---
title: Docker学习
date: 2020-05-27 22:32:16
tags:
categories:
- 框架
---

## 为什么用Docker

软件开发过程中常遇到的一个问题是，在自己机子上可以运行的程序，在别人那里不一定能运行，因为系统环境不一样。能不能让软件自带环境，在用户安装软件的时候连当初的系统环境一起安装了？这就是 **Docker** 要解决的问题。

## 与虚拟机的区别

VMware，VirtualBox 等虚拟机软件也可以在本机（host）上创建一个虚拟的环境，以便运行那些对系统有特殊要求的软件。

上述虚拟机软件目的是模拟整个操作系统，功能很强大，但也占用大量 host 资源。如果只是为了运行某一软件而安装一个操作系统，这显然有些过分了。

针对虚拟机的上述问题，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Container，即 LXC）。Docker 是当今最流行的 Linux 容器实现方案。

Docker 将应用程序和相应的依赖打包在一个文件中（image 文件），运行这个文件就启动了一个虚拟容器（container）。程序在这个虚拟容器中运行，虚拟容器可以隔离应用程序环境和 host 系统环境，这样就不用担心应用程序对系统环境的依赖问题了。

## 安装与启动

安装Docker

```
yum install docker
```
查看Docker版本

```
docker -v
```

启动与停止

```
systemctl start docker // 启动 docker
systemctl stop docker // 停止 docker
systemctl restart docker // 重启 docker
systemctl status docker // 查看 docker 状态
systemctl enable docker // 开机启动
```

## 镜像操作

列出镜像

```
docker images
```
搜索镜像

```
docker search image_name
```
拉取镜像

```
docker pull image_name
```
删除镜像

```
docker rmi image_name
```

## 容器操作

查看正在运行的容器

```
docker ps
```

查看所有容器

```
docker ps -a
```
创建容器常用的参数

```
创建容器命令：docker run
-i：表示运行容器。
-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
--name：为创建的容器命名。
-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。
-d：在run后面加上-d参数，则会创建一个守护式容器在后台运行。
-p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个－p做多个端口映射。
```

创建交互式容器

```
docker run -it --name=container_name docker.io/centos /bin/bash
```
创建守护式容器

```
docker run -id --name=container_name docker.io/centos
```
进入容器

```
docker exec -it container_name /bin/bash
```
退出容器

```
exit
```

停止容器

```
docker stop container_name
```
启动容器

```
docker start container_name
```
文件拷贝

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录 // 将文件拷贝到容器内
docker cp 容器名称:容器目录 需要拷贝的文件或目录 // 将文件从容器内拷贝出来
```
目录挂载

```
docker run -id -v 宿主机目录:容器目录 --name=container_name docker.io/centos --privileged=true
```

查看容器IP地址

```
docker inspect container_name
```
删除容器

```
docker rm container_name
```

## 备份与迁移

容器保存为镜像

```
docker commit container_name image_name

container_name // 容器名称
image_name // 新的镜像名称
```

镜像备份

```
docker save -o image_name.tar image_name

-o // 输出到的文件
image_name // 要打包的镜像
```

镜像恢复与迁移

```
docker load -i image_name.tar

-i // 输入的文件
image_name.tar 已经打包的镜像
```

## 参考

+ [Docker 简介](https://www.jianshu.com/p/b4488233e60b)
+ [Docker的简单介绍](https://zhuanlan.zhihu.com/p/142658407)


