---
title: 记一例排查 Emacs 显示颜色的问题
date: 2019-10-19 12:31:51
tags:
- Terminal
- Emacs
---

解决方案：

```bash
export TERM=gnome-256color
```

准确设置你的终端变量，不要保留用于兼容的 `xterm`，一般常见的终端 emacs 会认识的。`xterm` 会被当成白色背景。

<!-- more -->

我的桌面环境是 `KDE plasma`，使用的终端模拟器是 `konsole`，这台机器上的 `emacs` 没有更改任何颜色相关的配置。我观察到黑色背景的 konsole 下很多颜色对比度非常低，比如 minibuffer 的提示符，是一种非常暗的蓝色，在纯黑背景下极难看清。然而 ssh 连接上去看到的就不一样，就很舒服的颜色。

到这里，我认为是我的 konsole 颜色配置不太好。我换了十多种颜色主题结果都没解决。此时注意到颜色主题只能指定 8 种标准颜色，而我们的颜色显示丰富度更高，至少是 256 色。

尝试过程中，我试了下 konsole, konsole+tmux, iTerm ssh, iTerm ssh+tmux 几种组合，发现是否使用 tmux 对颜色显示有影响，konsole 下只要使用了 tmux，颜色也会正确。此时我开始从 tmux 内与外来考虑区别了。

很快发现 `$TERM` 这个环境变量会被 tmux 修改从而使 emacs 感知到环境变化。这个变量有以下几种常见的取值：

```bash
TERM=linux  # Linux 的 tty，比如 Ctrl-Alt-F2 切到的那种。
TERM=dumb  # Emacs 里面的 shell 或者 eshell 的取值，用来告知内部运行的程序，这不是真的终端。
TERM=xterm-256color  # 支持 256 色的终端，xterm 是一个被广泛接受的终端模拟器名称所以很多软件都会如此设置。比如 konsole, iTerm 甚至 WSL。当然也有真正的 xterm 本身。
TERM=screen-256color  # 为了向前支持，tmux 会假装自己是 screen。
TERM=gnome-256color  # 顾名思义。
TERM=konsole-256color  # 同上。
```

复现了一下，手动设置 `$TERM` 的取值就可以改变颜色，问题解决。

这个设定的原因是什么呢？在 emacs 中 求值 `(list-colors-display)` 你会得到一个显示所有 emacs 认为可以显示的颜色的 buffer，根据你的 `$TERM` 是否有 256color 后缀，它会判断你是否能支持 256 色。而这些颜色往往不会跟随终端主题变化，主题往往只改变基本的 8 种颜色。而前缀同样重要。如果前缀是 `xterm` 那么 emacs 认为你的终端背景是白色，于是就会给你比较暗的字符颜色。于是就 gg 了，而其他的变量会使 emacs 认为背景是黑色，which 是我们想要的结果。

也不是所有的前缀都被支持的，如果你设定了一个非标准的 `$TERM` 变量，emacs 将会拒绝工作。`$TERM` 设定为 dumb 时也一样。

问题最终的解决方案：konsole 设置中导出环境变量 `TERM=konsole-256color` 即可。
