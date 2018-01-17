---
title: 浙江大学 计算机视觉 复习提纲
date: 2018-01-14 19:18:40
tags:
- 课程
- 计算机视觉
---

﻿

- 计算机视觉
  - 计算题
  - 问答题
  - 推导题
  - 12-13道大题左右

## TODO List

- NMS 非极大值抑制
- LoG 的 G
- 一阶偏导**有限**差分
- Harris 推导
- PCA 协方差矩阵的用法
- weight 数

<!-- more -->

---------------------------------

## 格式塔法则 Gestalt Theory

- Law of Proximity 距离近的
- Law of Similarity 长得像的
- Law of Good Continuation 
- Law of Closure
- Law of Goodform
- Law of Figure / Ground

## Marr 视觉表示框架

### primal sketch

角点 边缘 纹理 线条 边界

### 2.5D sketch

在以观测者为中心的坐标系中

深度 法线 轮廓

### 3D model

在以物体为中心的坐标系中

恢复，重建和表示三维模型

## 二值图像
### 几何特性

- 面积 零阶矩
- 区域中心 一阶矩
- 方向
  - 最小化问题
  - 最小二乘法
- 伸长率 $E=\frac{\chi_{max}}{\chi_{min}}$
- 密集度 p 周长 A 面积 $C=\frac{A}{p^2}$
- 形态比：最小外接矩形的长宽比
- 欧拉数：连通分量数减去洞数

### 投影计算

### 连通区域

- 递归算法
- 序贯算法
- *区域边界跟踪算法*

## 边缘
>  基本思想 检测灰度不连续的地方
>  四种不连续
>  基于一阶和二阶的边缘检测
>  laplacian log算子
>  marr hildreth算子 为什么加G
>  canny 边缘检测 重点 要记住详细步骤

### 四种主要的不连续

- 深度不连续
- 颜色不连续
- 法向量不连续
- 光照不连续

### 基于一阶导数的边缘检测

**图像中用差分近似偏导数**

#### 梯度

$|G_{(x,y)}|\approx\max(|G_x|,|G_y|)$

#### Roberts 交叉算子

$$
G_x=
\begin{pmatrix}
1 &  0 \\
0 & -1 \\
\end{pmatrix}
G_y=
\begin{pmatrix}
0 & -1 \\
1 &  0 \\
\end{pmatrix}
$$

#### Sobel 算子

$$
G_x=
\begin{pmatrix}
-1 &  0 &  1 \\
-2 &  0 &  2 \\
-1 &  0 &  1 \\
\end{pmatrix}
$$

y 方向的同理，转置即可。

#### Prewitt 算子

速度较快

Sobel 的2全部换成1即可。

### 基于二阶导数的边缘检测

#### Laplacian 算子

$$
\nabla^2=
\begin{pmatrix}
0 & 1 & 0 \\
1 & -4 & 1 \\
0 & 1 & 0
\end{pmatrix}
$$



#### LoG 算子

高斯滤波 + 拉普拉斯边缘检测

- 平滑滤波使用高斯滤波
- 拉普拉斯算子计算二阶导数
- 边缘检测判据是二阶导数交叉零点是一阶导数较大峰值
- 使用线性内插方法在子像素分辨率水平上估计边缘的位置

草帽算子

两种等效方法

- 图像与高斯函数卷积，再求卷积的拉普拉斯微分
- 求高斯函数的拉普拉斯微分，再与图像卷积

使用高斯滤波器的原因：

平滑去噪和边缘检测是一对矛盾，应用高斯函数的**一阶导数**，在二者之间获得最佳的平衡

### Canny 边缘检测

步骤：

- 高斯滤波 平滑图像
- 一阶偏导有限差分 计算 梯度幅值与方向
- 非极大值抑制
- 双阈值算法检测和连接边缘

## harris角点检测
> 知道原理
> 会推导公式 约等于的 特征值

原理：

一个窗口在图像上移动。平滑区域里，在各个方向上都没有变化；边缘上，在边缘方向没有变化；角点，在各个方向都有变化。

公式：

窗口平移产生的变化：$E(u,v)\approx [u,v]M[u,v]^T$

$E(u,v)=\sum_{x,y}w(x,y)[I(x+u,y+v)-I(x,y)]^2$

$u,v ~\text{are small in values}$

$I(x+u,y+v)\approx I(x,y)+uI_x(x,y)+vI_y(x,y)$

$E(u,v)=\sum_{x,y}w(x,y)[I(x+u,y+v)-I(x,y)]^2$

$=\sum_{x,y}w(x,y)[I(x,y)+uI_x(x,y)+vI_y(x,y)-I(x,y)]^2$

$=\sum_{x,y}w(x,y)[uI_x(x,y)+vI_y(x,y)]^2$

$$
=[u,v]\sum_{x,y}w(x,y)
\begin{pmatrix}
I_x^2 & I_xI_y \\
I_xI_y & I_y^2 \\
\end{pmatrix}
[u,v]^T
$$

$$
M=
\sum_{x,y}w(x,y)
\begin{pmatrix}
I_x^2 & I_xI_y \\
I_xI_y & I_y^2 \\
\end{pmatrix}
$$

其中 $I_x,I_y$ 是矩阵 $M$ 的特征根。

上面那句没懂。

$R=\det M-k(\text{trace}~M)^2$

$\det M=\lambda_1\lambda_2$

$\text{trace}~M=\lambda_1+\lambda_2$

算法流程：

- 找出大于阈值的 R 值
- 选取其附近的局部极大值

## SIFT 描述子的计算

> 为何只使用梯度信息
> 计算的基本步骤 full version
> 如何实现旋转不变性
> 尺度不变的原理

### 旋转不变性

因为旋转的时候每一个关键点周围的点也会跟着旋转，不会影响SIFT向量。

### 尺度不变性

图像金字塔

### 计算步骤

- Scale-space extrema detection
  - uses Difference-of-Gaussian function
- Keypoint localization
  - subpixel location and scale fit into the model
- Orientation assignment
  - 1 or more for each point
- Keypoint descriptor
  - from local image gradients

## Hough 变换 直线检测

基于投票，流程如下：

- 适当地参数化向量空间
- 假定参数空间每个单元都是累加器
- 对图像空间上每一点，累加器++
- 选取最大值

## 图像的傅立叶变换
> 变换本身和性质不用记
> 理解图像的低频与高频成分 能通俗解释
> 理解拉普拉斯金字塔的每一层是带通滤波，是怎么来的
> 语言解释，不需要公式
> 从高斯金字塔来的

拉普拉斯金字塔：高通减低通

低通：变化慢的信息

高通：变化快的信息

## 相机模型
> 理解 光圈 景深 焦距 视场

- 光圈越大~~越土豪~~，景深越小
- 焦距越大，视场越小

Thin lens equation:

$\frac{1}{d_o}+\frac{1}{d_i}=\frac{1}{f}$

视场角：$\text{AFOV}^\circ=2\tan^{-1}(\frac{h}{2f})$

上面的 $h$ 是什么？

TODO 相机模型的图

### 相机参数在成像各个阶段的作用

- 世界坐标系 => 相机坐标系：外参数
- 相机坐标系 => 像平面坐标系(2D)：内参数
- 非理想模型：=> 像素：畸变参数
  - $k$ 和径向畸变有关
  - $p$ 和切向畸变有关

## 理想针孔相机模型
> 基本投影公式 画图说明 齐次坐标形式的透视投影公式 矩阵形式
> 内参和内参矩阵 不包括畸变参数 会写会背即可

Pinhole camera model:

$-x=f\frac{X}{Z}$

## 径向畸变和切向畸变
> 常见的哪两种 各是什么原因引起的
> 外参有哪几个，含义
> 内参 外参 畸变参数在成像各个阶段的角色
> 三维物体到真实图像的过程



## 相机标定
> 需要求解哪些参数
> 基于pattern / reference object 的相机定标
> 已知什么，求解什么
> 简述基本过程，几个步骤

### 相机参数

- 4 个内参 $(f_x,f_y,c_x,c_y)$
- 6 个外参 $(\theta,\varphi,\psi,t_x,t_y,t_z)$
- 5 个畸变参数 $(k_1,k_2,p_1,p_2,k_3)$

内参矩阵：
$$
M=
\begin{pmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1 \\
\end{pmatrix}
$$


### 过程

- 已知
  - N 个角点的标定对象
  - K 个标定对象的视角
- 求解
  - 相机参数，包括内参，外参与畸变参数
- 流程
  - 标定对象：知道网格角点的位置
  - 从图像中找到角点
  - 建立等式：将图像坐标转换到世界坐标的等式
  - 求解，得到相机参数

## 立体视觉 三角测量基本原理
> 会画视差disparity的图 并能推导公式
> 立体视觉的基本步骤

### 基本步骤

- 标定相机，消除畸变
- 校正图像
- 计算差距
- 估计深度

## 三维数据获取 结构光成像系统的构成

> 利用结构光获取三维数据的基本原理
> 会画图 会推导公式
> icp算法的作用和基本步骤

TODO 画图与推导公式

### 结构光成像系统

- 结构光投影仪
- CCD 相机
- 深度信息重建系统

### ICP 算法

迭代最近点算法 Iterative Closest Point

给定两个三维点集 X 与 Y，将 Y 配准到 X。

- 计算 Y 中每一个点在 X 中的对应最近点
- 求使上述对应点对的平均距离最小的**刚体变换**，获得刚体变换参数（平移参数与旋转参数）
- 对 Y 应用上一步求得的刚体变换（平移与旋转），更新Y
- 如果不到阈值一下，从第一步重新迭代

## 光流

> 解决什么问题
> 三个基本假设
> 一个点的约束公式 会推导
> 哪些位置的光流比较可靠，why

解决像素对应问题，找到两幅图之间距离不远的像素之间的对应关系。

基本假设：

- 亮度恒定 Brightness constancy
- 空间相干性 Spatial coherence
- 小移动 Small motion

### 一个点的约束公式

$$
O\approx I_t+\nabla I\cdot[u,v] \\
\text{证明如下：} \\
O=I(x+u,y+v)-H(x,y) \\
\approx I(x,y)+uI_x+vI_y-H(x,y) \\
\approx (I(x,y)-H(x,y))+uI_x+vI_y \\
\approx I_t+\nabla I\cdot[u,v]
$$



## 图像拼接
> 实现两张图像自动拼接的基本步骤

- 找到关键点
- 建立 SIFT 描述子
- 建立一一对应关系
  - 计算 SIFT 描述子之间的欧氏距离
- 拟合变换矩阵
  - 仿射变换
- RANSAC

### RANSAC 随机抽样一致性算法

> 理解其过程的核心思想
> 优点，基本步骤

*RANdom SAmple Consensus*

#### 优点

#### 步骤

- ​

## 人脸识别 主成分分析
> PCA 的基本思想 作用
> 优化目标函数的推导
> a1TSa1

### PCA 主成分分析

#### 基本思想 作用

降维

#### 优化目标函数的推导

投影方向 $\vec a_1$，有 $a_1^Ta_1=1$

$d$ 维空间中 $\vec x$
最大化 $\text{var}(z_1)=\text{var}(\vec a_1\cdot \vec x)$

求投影方向，即 $\arg\max _{a_1}\text{var}(z_1)$

$\text{var}(z_1)=a_1^TSa_1$

其中 $S=E(x_i,y_i)-E(x_i)E(y_i)=\text{Cov}(x_i,y_i)$

使用 Lagrange 乘子法

记 Lagrange 乘子为 $\lambda$

转化为最大化 $a_1^TSa_1-\lambda(a_1^Ta_1-1)$

对 $a_1^T$ 求微分并且令结果为0，得

$Sa_1-\lambda a_1=0$

此为最优化的必要条件。

上式就是矩阵特征值的定义，所以必须用协方差矩阵最大特征值对应的特征向量，转化为求协方差矩阵。

### eigenface
> 是什么，基本步骤
> 将重构用于人脸检测的原理

#### 步骤

- 将所有人脸归一化
- 通过 PCA 计算获得一组特征向量，一般 100 个就足够了
- 将每个人脸投影到此空间中，得到坐标
- 对于输入的图像，度量在此空间中的某种距离得到最近的结果

## visual recognition
> 基本任务 4类
> 都有哪些挑战因素



## 基于 Bag of Words （词袋）的物体分类
> 是什么意思，几个基本步骤

- 特征提取与表示
- 通过训练样本聚类来表示字典(codebook)
- 以字典的直方图描述图像
- 以 BoW 来分类未知图像

## 物体识别 CNN

计算参数个数与连接数

不要忘记 bias 项？

- 参数个数
  - 卷积核面积 $\times$ 卷积核个数
- 连接数
  - 参数个数 $\times$ feature map 面积
- 权重 (weight) 数
  - 同连接数

## 图像分割

### 图像分割的目标

将像素集合转换成有意义的或者感觉上相似的区域

### 基于聚类的语义分割

基于 `K-means` 聚类

聚类之前先 SIFT

聚类之后要构建

### 基于 mean shift 的图像分割

要求掌握：基本原理与基本思路

不需要具体步骤

------------------

# Additional Part

## 计算机视觉的研究内容

- 输入设备
- 低层视觉
- 中层视觉
- 高层视觉
- 体系结构