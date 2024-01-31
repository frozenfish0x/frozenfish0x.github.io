---
title: docker
date: 2023-12-17 19:08:03
tags:
---

# 1.docker

```
Docker 通常用来做什么
应用分发、部署，方便传播给他人安装。特别是开源软件和提供私有部署的应用
快速安装测试/学习软件，用完就丢（类似小程序），不把时间浪费在安装软件上。例如 Redis / MongoDB / ElasticSearch / ELK
多个版本软件共存，不污染系统，例如 Python2、Python3，Redis4.0，Redis5.0
Windows 上体验/学习各种 Linux 系统
```
<!--more-->


# 2.安装&使用

```shell
sudo pacman -S docker
 sudo systemctl start docker
sudo systemctl enable docker

测试
sudo docker run hello-world
```



# 普通人使用需要掌握知识：

docker-comple什么的他是docker的一件启动容器功能。这个软件可以一件话快速部署容器，并且出现问题后可以立刻使用此软件立刻创建其他容器。

docker需要固定持久化的目录。用来存储数据。

docker拉去镜像 

```
docker pull
```

docker 查看容器

```
docker ps
```

docker 查看镜像

```
docker image
```

docker 删除容器

```
docker rm 名字或id
```

docker创建虚拟网络

```
docker network create 具体参数请查阅文档
```

docker每个容器内部环境都不同，独立隔离。docker有内部端口和外部端口映射这么一说。docker十分方便。文件存储在哪。即使挂了重启挂在同杨目录也能起来服务





docker 不是虚拟机。完全不一样的底层，



问题，今天我在使用docker安装openwrt时出现虚拟化的环境影响到宿主，十年老开发表示这属于很怪的问题，

我重启后，init=/bin/bash绕过system直接启动bash，停止了docker服务。并关闭了docker的自启动

systemctl stop docker

systemctl disable docker

然后docker有个命令是查看他的目录。我进去删掉其中的文件。于是就不在自弃，然后顺利删除容器。

这种docker我刚学就遇到这么棘手的问题。实在不想花时间在这上面排查。所以我决定解决问题。而不是排查问题。如了解docker启动原理。核心。底层原理。这不是我的排查范围，我想着破坏docker的文件导致他不能自启动。后续在进行删除，也能保全docker其他的东西。1



群友说：debian虚拟化更成熟。arch不适合在生产环境种运行。、

docker逃逸。感觉很像我这个问题
