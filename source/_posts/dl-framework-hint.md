---
title: 深度学习框架坑点槽点笔记
date: 2017-09-25 15:52:33
tags:
- Python
- 编程
- 深度学习
---

PyTorch 当中`conv2d`不存在 valid 与 same 两种模式的选择，默认为 valid。可能他们觉得应该手动 padding 比较合理吧。