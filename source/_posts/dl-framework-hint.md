---
title: 深度学习框架坑点槽点亮点注意点笔记
date: 2017-09-25 15:52:33
tags:
- Python
- 编程
- 深度学习
---

踩到啥就说啥。

<!-- more -->

`PyTorch` 当中`conv2d`不存在 `valid` 与 `same` 两种模式的选择，默认为 `valid`。可能他们觉得应该手动 `padding` 比较合理吧。

`Keras` 的 `conv2d` 默认 `padding` 方式为 `valid`。

`PyTorch` 的`deconv2d` 默认为 `same` 工作模式，并且设置了长度为1的 padding 之后也没有起效果（待查）的样子？总之要注意。

`Keras` 使用后端函数的东西无法直接放进模型，需要用 `Lambda` 层包装一下。为了统一抽象的代价吧。

`Keras` 模型有 `summary` 函数，可以打出模型之中每一层的参数总个数(但是不包括shape以及层相关的信息，比如卷积层的padding，strides等等)，还可打出输出张量的 Shape，PyTorch 可以直接 print 一个模型，得到每一层的参数但是不包括输出的 Shape。你们俩让我怎么吐槽比较好（

`OpenCV` 官方文档：http://docs.opencv.org/master/

`Keras` 英文文档：https://keras.io/

`Keras` 中文文档：http://keras-cn.readthedocs.io/en/latest/

cv2.imwrite 写入灰度图像再读出时候会自动转换为三通道的…… http://docs.opencv.org/master/d4/da8/group__imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56

