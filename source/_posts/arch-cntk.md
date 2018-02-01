---
title: Arch Linux 中 CNTK 安装后运行时报错
date: 2018-02-01 21:46:13
tags:
- Linux
---

如果报错信息如下，原因是 `openmpi` 配置不正确。

~~还是第一次见 DL 框架要使用 MPI 的，感觉不错，就是这依赖的文件不太妙~~

```
ImportError: libmpi_cxx.so.1: cannot open shared object file: No such file or directory
```

嘛，官方只钦定 `Ubuntu` 那我们自己修库的依赖：

```sh
sudo pacman -S openmpi
sudo ln -s /usr/lib/openmpi/libmpi_cxx.so /usr/lib/libmpi_cxx.so
sudo ln -s /usr/lib/libmpi_cxx.so /usr/lib/libmpi_cxx.so.1
sudo ln -s /usr/lib/openmpi/libmpi.so /usr/lib/libmpi.so
sudo ln -s /usr/lib/libmpi.so /usr/lib/libmpi.so.12
```

虽然很脏，并且我还没看其他平台比如 `RHEL` 上面的 `OpenMPI` 库是不是单独分了目录。总之这样 import 的时候没出错，有问题我再更新博客吧。