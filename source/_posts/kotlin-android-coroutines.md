---
title: Kotlin Android 开发当中使用 coroutines 实现异步操作
date: 2017-10-08 01:21:12
tags:
- 编程
- Android
- Kotlin
---

## 背景

本文写作于 2017年10月8日，kotlin 语言版本为 1.1，其中协程为实验特性。需要指定编译器选项以启用此功能。实验阶段表明此功能稳定性不能得到保证，请慎用！

Kotlin 官网上对于协程实现的大概介绍：https://kotlinlang.org/docs/reference/coroutines.html

教程实例 Repo：https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-android/example-app/app/src/main/java/com/example/app/MainActivity.kt

kotlinx.coroutines 协程库：https://github.com/kotlin/kotlinx.coroutines

<!-- more -->

## 概述

协程可能是近几年来最新的并行/并发模型实现，常见语言中最早的实现可能是`lua`，近年来新晋语言`golang`的崛起更把这个概念发扬光大。

Kotlin 的实现设计思路有以下几个特点：

- 尽可能不使用核心语言，而是标准库来实现。
- 标准库只提供核心功能，官方的第三方库提供应用层接口。
- 异步等操作使用协程封装，同时协程可用于其他方面尽可能替代线程。

尽管如此，官方也不得不加入`suspend`关键字来标记协程可挂起函数，同时协程的实现也需要编译器支持。

`suspend`关键字与`C#`等语言中的`async`关键字很接近。线程的调度由操作系统完成，而协程需要由运行时管理。为了标记一个函数可以被运行时挂起，需要此关键字。

## 实现

未完待续