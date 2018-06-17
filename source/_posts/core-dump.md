---
title: 关于 core dump 的那些事
date: 2018-06-17 11:02:09
tags:
- Linux
- 编程
---

本文的目标是记录下来关于 `core dump` 你应该知道和操作的那些事情。

施工中。

<!-- more -->

## core dump 是什么

`core dump` 这一词的来源是 `Magnetic core memory` 磁芯存储器中的 core，而不是我曾经认为的和 kernel 相关的那个含义。磁芯存储器是五十多年前古时候的内存了，可见这一 term 的悠久历史，也说明核心转储文件（自然这个翻译也不甚靠谱了）本质只是记录了事故现场的内存快照。

## 如何获取 dump 文件

### 如何产生 core dump 文件

主要的来源是用户态程序崩溃时 kernel 自动帮你存储的，这也是一般情况下最常见和有意义的来源。对于内核来讲，红帽有个黑科技在内存中运行两份内核，kernel panic 时候备用内核接管，把主内核 dump 出来。当然 gdb 等调试器也可以给跑得好好的进程直接做个 dump 出来。

```sh
gdb -p <pid>
gdb>generate-core-file
```

会生成文件名为 `core.<pid>` 的 core dump 文件。

下一个问题是 kernel 如何设置是否进行 core dump 以及 dump 的结果如何处理，存放在何处。这些通过系统暴露出来的 `/proc` 文件系统进行设置。

### “吐出来的核” 存放在哪里

## 如何进行分析

最主要的方式还是 `gdb`，同时有一个名为 `crash` 的小工具可以帮助我们。

## 参考资料

https://en.wikipedia.org/wiki/Core_dump

https://en.wikipedia.org/wiki/Magnetic-core_memory

https://wiki.archlinux.org/index.php/Core_dump