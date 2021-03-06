---
title: 浙江大学 计算机图形学 课堂笔记
date: 2017-09-18 09:48:52
tags:
- 计算机图形学
- 课程
---

童若锋老师的计算机图形学课程

- e-mail: trf at zju
- ftp://csyq2017:csyq2017@give.zju.edu.cn
- [http://give.zju.edu.cn/cgcourse](http://give.zju.edu.cn/cgcourse)
- TA-email: 
- Evaluation
  - Assignment 30%
  - Final-Exam Big-Project 40%
  - In-class performance 30%
    - in-class test
    - attendance
    - question-answer

<!-- more -->

## 第一课

### 概要

- 计算机生成的图像 - 特点
  - 可以变换视角
  - 计算机生成的是三维模型，经过渲染成为像素化的标量图片（先尽量根据现实建模，再“拍照”）
  - 对时效有要求，需要做到实时渲染

### 课程目标

- 计算机图形学要解决的基本问题
- 解决这些问题主要的原理与方法
- 基本编程技能
- 获取自己学习，解决问题的能力
- 课程假定具有基本的使用 `OpenGL` 的能力，或者说，不以此为重点

### 介绍

- 使用像素(pixel - 最小的显示单元)图片给人真实的体验(looks real)
- recommended - 将世界中的物体使用计算机建模，处理并且显示的科学
  - 3D object ===> image
  - display rendering
  - image: aray of pixels (RGB)
  - graphics: object (shape, material, ...)
- 三个基本任务
  - 建模
  - 模拟物体的行为
  - 显示

### 如何表示一个图像(物体)

使用怎样的数据结构比较合适？

- 点 `Point3D { double, double, double }`
- 矩形呢？
  - `cuboid { LCS3D local; double x; double y; double z; }` 与四个点等价。本地坐标系需要9个参数。
- 任意物体呢？哪种抽象最佳？
  - 顶点集合
  - 边线段集合
  - 点云
  - 表面平面图形集合
- 要解决的问题
  - 水如何流动起来？
  - 弹性，相对运动？
  - 光照效果？

## 第二课

### 视觉计算领域

- 图像处理
  - 图像 -> 图像
  - 图像 -> 区域
- 模式识别
  - 图像 -> 符号化描述
- 计算机视觉
  - 图像 -> 图形
- 计算机图形学
  - 图形 -> 图形
  - 图形 -> 图像

### 光栅化(扫描转换)

在二维空间当中如何画线段？

空间是像素点的。将抽象的数学直线画进去。要越快，越均匀，越接近越好。给出线段两个端点的坐标。

判断更接近于哪个坐标轴，然后以一个像素为步长增加，计算另一侧坐标(浮点加法！)然后四舍五入到整数像素点。这个方法为 DDA (Digital Differential Analyzer) 算法。

这个思路并非最优，cutting-edge 的做法是 Bresenham 算法。

> **Bresenham算法的思想是将像素中心构造成虚拟网格线，按照直线起点到终点的顺序，计算直线与各垂直网格线的交点，然后根据误差项的符号确定该列像素中与此交点最近的像素。**

进行优化的思路：浮点运算 -> 整型运算    乘法运算 -> 加法运算

参考文章： [https://zhuanlan.zhihu.com/p/20213658](https://zhuanlan.zhihu.com/p/20213658)

## 第三课

### 扫描转换 - 画圆

圆周上取点，再用线段连起来？

如何取点，直角坐标系等距取点不均匀 => 使用极坐标系

利用对称性，计算八分之一个圆弧即可。

$$x_i=x_c+r\cdot\cos(i\cdot\Delta\theta)$$

$$y_i=y_c+r\cdot\sin(i\cdot\Delta\theta)$$

三角函数开销极大，如何加速运算？

同样的思想：增量算法

$$x_{i+1}=r\cdot\cos(\theta_i+\Delta\theta)$$

和角公式展开：

$$x_{i+1}=x_i\cos\Delta\theta-y_i\sin\Delta\theta$$

实际使用画圆的 Bresenham 算法

其实也可以 look up table

### 不同表示曲线的方式

- explicit curve               $y=f(x)$
- parametric curve        $x=x(t),y=y(t)$
- implicit curve               $g(x,y)=0$


参数曲线可以表达函数之外的任意曲线，而且遍历参数$t$即可描出整条曲线。故在计算机图形学之中应用最多。

隐式曲线不一定能转换为显式表达式，例如：$x^y\sin x+y^x\cos\ln x=0$

解决思路：找出一个解，假设它连续，delta 法描出。

### OpenGL / GLUT

- 实验内容全部使用 `OpenGL` 完成，版本不详。

如何填充颜色为渐变效果？trick：不同顶点设置为不同颜色。

### 多边形填充

如何判断像素在多边形内？

顶点表示 => 像素格点表示

有多种方法，较合理而常用的有：

- 到所有顶点转角之和：内部为360度，外部为0。
- 奇偶校验：射线向外，穿过边次数奇数次在内部，偶数次在外部。
  - 水平，竖直的射线计算量更小。
  - 一条线上面的点可以复用之前的计算结果。

## 第四课

### 填充多边形

- 如何按照线填充多边形时求出横/竖线与边缘的交点？

> 计算交点是计算机图形学之中最不喜欢做的事情。

使用 DDA 算法求交点 - 扫描线算法

*图像处理：chain coding / contour following*

判断每个点是否在多边形内计算量过大。

填充的计算量和画边界的计算量基本没有区别。

还有一个问题：计算需要绘制的点的颜色。(shading)

- 给出图像像素边框以后求填充 - 种子填充算法

### Aliasing 走样(锯齿)

#### 问题

- 从曲线映射到离散的最小单位：像素点
- 纹理相关
- 线过细，落不在像素点中心导致整个消失

#### 解决方案

- 超采样
- 区域采样（算百分比）

### Clipping 裁剪

- Cohen-Sutherland Line Clipping
  - 区域编码算法
    - 两个端点均编上四位区域编码，然后求按位与。若结果为0则需要计算交点。
    - 还有其他情况。
- 多边形裁剪
  - 使用窗口与多边形的每一条边进行裁剪。

## 第五课

### 几何变换与坐标变换

- 平移
- scaling 变比变换 缩放
  - 可以完成镜像等操作
  - 一般假设缩放中心在原点
- 旋转
- shearing 不知道中文叫什么的变换

均可以用左乘一个矩阵来实现

对于二维的坐标，如何完成平移变换？ => 乘三阶方阵，就可以相当于加上一个常数项。

- Homogeneous Co-ordinates 齐次坐标

三维情况同理

矩阵表示变换方便连乘

## 第六课

### OpenGL API

版本不详，固定管线时代的 API

{% asset_img gl-wiki.svg %}

共有三个矩阵堆栈，使用 `glMatrixMode` 指定。

- GL_MODELVIEW
- GL_PROJECTION
- GL_TEXTURE

相关的 API，关于 CTM (当前变换矩阵)

m 为16维向量，表示 4*4 矩阵。

```c
glMatrixMode();

glLoadIdentity();
glLoadMatrix{fd}(*m);

glMultMatrix{fd}(*m);

glTranslate{fd}(x, y, z);
glScale{fd}(x, y, z);
glRotate{fd}(angle, x, y, z);
```

注意，OpenGL 当中调用变换 API 的顺序和乘法进行的顺序是反的。

### Viewing

世界坐标系与观察坐标系

2D 视口变换

3D 变换

Projection

- 平行投影
- 透视投影

## 第七课

--------------------------------

## 冬学期

## 第一课

### 消除被遮挡的物体 - 消隐

- 图像空间 - 遍历图像中的每一个像素，决定如何绘制这个像素 => Z-buffer，最实际的算法
- 物体空间 - 删掉被遮挡的物体


### Back-face Culling

判断法线

约定 - 右手大拇指指向外法线方向

### 画家算法

从后往前画，前面的自然会覆盖后面的。

互相遮挡？ => 分割区域。

#### Warnock's Algorithm

- 对区域分割，如果可以排序就停止，否则递归地分割下去。

#### Z-buffer Algorithm

- F-buffer => color value
- Z-buffer => Z value