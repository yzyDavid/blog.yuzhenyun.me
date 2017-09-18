---
title: 浙江大学 计算机图形学 课堂笔记
date: 2017-09-18 09:48:52
tags:
- 计算机图形学
- 课程
---

童若峰老师的计算机图形学课程

- e-mail: trf at zju
- ftp://csyq2017:csyq2017@give.zju.edu.cn
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
- 三个基本任务
  - 建模
  - 模拟物体的行为
  - 显示

### 如何表示一个图像

使用怎样的数据结构比较合适？