---
title: ibverbs Man page 翻译
date: 2019-11-05 23:03:39
tags:
- Infiniband
- RDMA
---

翻译一些读过了的 `ibverbs` API。

<!-- more -->

### ibv\_get\_device\_list

### ibv\_free\_device\_list

获取或者释放一个 `RDMA` 设备列表。获取时候传入的整型指针可空，如果非空，将会传出设备的个数。

### ibv\_open\_device

### ibv\_close\_device

打开或关闭设备上下文。

### ibv\_alloc\_pd

### ibv\_dealloc\_pd

`PD` 为 `protection domain`

### ibv\_reg\_mr

### ibv\_dereg\_mr

`MR` 为 `memory region`

每个 `MR` 关联在一个 `PD` 上。

### ibv\_create\_cq

### ibv\_destroy\_cq

`CQ` 为 `completion queue`

### ibv\_create\_qp

### ibv\_destroy\_qp

`QP` 为` `queue pair`

每个 `QP` 关联在一个 `PD` 上。
