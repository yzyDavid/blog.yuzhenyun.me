---
title: Docker Hello World
date: 2017-11-07 21:53:59
tags:
- docker
---

本文记录第一次使用 `Docker` 的操作，以备日后查阅。

原文链接：https://docs.docker.com/get-started/

<!-- more -->

## 安装 Docker

系统相关，源里都有，不说废话。

cheat sheet :

```bash
sudo yum remove docker{,-engine,-common,-selinux}
sudo yum install yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce
docker run hello-world
docker --version
docker run dockersamples/visualizer  # for no swarm running, nothing can be seen
```

NFS 共享根文件系统的几台机器之中只能有一台启动 docker，如何绕过此限制？

## 基本使用

```bash
docker images
docker container ls
docker run -it ubuntu bash
```

在 CentOS 中跑起来的 ubuntu 镜像 bash，运行 uname -a 结果如下。可见内核版本仍然为 CentOS 的版本。

Linux 1587622a5387 3.10.0-693.2.2.el7.x86_64 #1 SMP Tue Sep 12 22:26:13 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux

## Docker 集群

```bash
docker swarm init --advertise-addr 10.*.*.*  # IP 隐去，请填写一个本机IP
docker service create \
--name=viz \
--publish=8088:8080/tcp \
--constraint=node.role==manager \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
manomarks/visualizer  # 我现在还看不懂，创建可视化工具用的
docker service ls
docker service rm [service ID]
```

## Dockerfile

本部分中我们自己创建一个 docker 镜像。

docker 镜像基于已有的上游镜像，在基础上自己添加内容。

创建 Dockerfile app.py requirements.txt 三个文件，内容按照教程原样复制：

https://docs.docker.com/get-started/part2/#apppy

建立镜像的命令：

```bash
sudo docker build -t friendlyhello .
```

## Docker Hub

```bash
docker login
docker tag [imagename] [username]/[repo]:[tag]
docker push [username]/[repo]:[tag]
```

