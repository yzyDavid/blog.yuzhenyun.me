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
```

