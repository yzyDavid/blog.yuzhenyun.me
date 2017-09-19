---
title: git-blame
date: 2017-09-19 10:24:45
tags:
- git
- 编程
---

某些小朋友在 `git` 托管的文件里面聊起了天我不能忍啊……

这个时候请你们使用

```bash
git blame
```

 如果使用`zsh`的 alias 的话，可以直接使用`gbl`这个缩写。

<!-- more -->

```bash
which gbl    
gbl: aliased to git blame -b -w
```

```bash
gbl README.md # 将会输出文件的每一行由谁改动
```

