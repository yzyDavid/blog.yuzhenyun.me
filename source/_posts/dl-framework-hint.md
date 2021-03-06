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

`OpenCV` 官方文档：[http://docs.opencv.org/master/](http://docs.opencv.org/master/)

`Keras` 英文文档：[https://keras.io/](https://keras.io/)

`Keras` 中文文档：[http://keras-cn.readthedocs.io/en/latest/](http://keras-cn.readthedocs.io/en/latest/)

cv2.imwrite 写入灰度图像再读出时候会自动转换为三通道的……问题应该是imread读入时候转换的。Keras 的 image.load_img 函数同样有这个问题，需自行转换。

```python
img_a = np.zeros((x, y), dtype=img.dtype)
img_a[:, :] = img[:, :, 0]
```

`PyTorch` 英文文档：[http://pytorch.org/docs/master/](http://pytorch.org/docs/master/)

`PyTorch` 中貌似不存在 argmax 函数，我们可以使用 `numpy` 实现这种功能。np.argmax 返回 Python 整数。`PyTorch` Tensor 可以调用 .numpy() 方法将其转换为 `numpy` 数组。