---
title: 深度学习框架坑点槽点笔记
date: 2017-09-25 15:52:33
tags:
- Python
- 编程
- 深度学习
---

`PyTorch` 当中`conv2d`不存在 `valid` 与 `same` 两种模式的选择，默认为 `valid`。可能他们觉得应该手动 `padding` 比较合理吧。

`Keras` 的 `conv2d` 默认 `padding` 方式为 `valid`。

`PyTorch` 的`deconv2d` 默认为 `same` 工作模式，并且设置了长度为1的 padding 之后也没有起效果（待查）的样子？总之要注意。