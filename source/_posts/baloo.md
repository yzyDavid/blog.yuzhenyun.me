---
title: KDE plasma 5 的 baloo 文件搜索
date: 2018-01-03 01:05:12
tags:
---

垃圾应用，磁盘IO一直高占用。

iotop 中可见一直在读硬盘。关闭即可：

```sh
sudo balooctl stop
sudo balooctl disable
```

KDE system settings:

搜索 -> 文件搜索 -> 禁用

世界清净了

KDE 重新变得美好！

(滚去写 java 了)