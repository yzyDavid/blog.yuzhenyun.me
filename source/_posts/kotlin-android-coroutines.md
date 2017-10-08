---
title: Kotlin Android 开发当中使用 coroutines 实现异步操作
date: 2017-10-08 01:21:12
tags:
- 编程
- Android
- Kotlin
---

## 背景

本文写作于 2017年10月8日，`kotlin` 语言版本为 1.1，`kotlinx.coroutines`版本为 0.19.1，其中协程为实验特性。需要指定编译器选项以启用此功能。实验阶段表明此功能稳定性不能得到保证，请慎用！

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

`suspend`关键字与`C#`等语言中的`async`关键字很接近。线程的调度由操作系统完成，而协程需要由运行时管理。为了标记一个函数可以被运行时挂起，需要此关键字。当然协程的范围比异步更广，这里不再赘述。

## 实现

本部分中我们通过完成`kotlinx.coroutines`官方教程来实际体验一下`kotlin`的协程。https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md 这篇文档为`kotlinx`协程的 tutorial，我们主要完成 https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md 这一篇专注于 UI 编程的 tutorial。首先可以 clone https://github.com/Kotlin/kotlinx.coroutines 这个 repo，其中 ui/kotlinx-coroutines-android/example-app/ 目录下有 tutorial 用到的脚手架。

使用 Android Studio 打开项目，跟着向导一通安装 SDK 之后可以通过 build 了，来看一下 MainActivity.kt 目前的内容：

```kotlin
package com.example.app

import android.os.Bundle
import android.support.design.widget.FloatingActionButton
import android.support.v7.app.AppCompatActivity
import android.view.Menu
import android.view.MenuItem
import android.widget.TextView
import kotlinx.android.synthetic.main.activity_main.*
import kotlinx.android.synthetic.main.content_main.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)
        setup(hello, fab)
    }

    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        // Inflate the menu; this adds items to the action bar if it is present.
        menuInflater.inflate(R.menu.menu_main, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        val id = item.itemId
        if (id == R.id.action_settings) return true
        return super.onOptionsItemSelected(item)
    }
}

fun setup(hello: TextView, fab: FloatingActionButton) {
    // placeholder
}
```

### 在 UI 中使用 coroutine

向文件头部添加如下 import：

```kotlin
import kotlinx.coroutines.experimental.android.UI
import kotlinx.coroutines.experimental.delay
import kotlinx.coroutines.experimental.launch
```

将 setup 函数替换为：

```kotlin
fun setup(hello: TextView, fab: FloatingActionButton) {
    launch(UI) {
        for (i in 10 downTo 1) {
            hello.text = "Countdown $i ..."
            delay(500)
        }
        hello.text = "Done!"
    }
```

运行效果图如下：

{% asset_img 1.png %}

如何做到点击右下角圆形悬浮按钮就停止倒计时呢？在前面的 launch(UI) { ... } 随后的 lambda 表达式后链式调用以下内容：

```kotlin
.apply {
        fab.setOnClickListener { cancel() }
    }
```

随后刷新，调试。可以看到点击后倒计时停住不动了。

Standard.kt 中定义的 `let` `apply`等 one-liner 工具函数的说明可以查阅文档：https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html

在 UI 上下文中可以使用 actor 完成进阶功能，比如限制线程上同时存在的协程个数。这些需求比较高级，需要使用时请查阅文档即可。

### 在非 UI 上下文中执行阻塞耗时的操作

这种操作主要有两种，一是计算密集型，占用大量 CPU 时间使得函数无法及时返回，造成阻塞的。本文将要举的例子是 Fibonacci 数列的计算。还有一种是 IO 密集型，比如最常见的网络访问。Android 系统是禁止 UI 线程访问网络的，这样的尝试会被强行抛异常。下面介绍一下具体做法。

普通的 fib 函数：

```kotlin
fun fib(x: Int): Int = if (x <= 1) 1 else fib(x - 1) + fib(x - 2)
```

特效版本的 fib 函数：

```kotlin
import kotlinx.coroutines.experimental.run
import kotlinx.coroutines.experimental.CommonPool

suspend fun fib(x: Int): Int = run(CommonPool) {
  if (x <= 1) 1 else fib(x - 1) + fib(x - 2)
}
```

可见我们只要显式指定协程池为 CommonPool 即可。这里的 run 函数比起 launch 来，区别在于可以返回值而非 Unit。

### async/await

这部分内容没有包含在原文当中，但是对于了解过其他语言中这种编程范式的读者来说很有必要也很易懂。

大概用法就是：

```kotlin
val job = async { ... }
val result = job.await()
```

运行环境等等问题和前文一样啦。仍然可以指定运行的上下文。

## 总结

和本人较为熟悉的 C#/Javascript 相比起来，语法噪音稍稍多了一点点，但是带来的掌控性与灵活性，还有协程实现本身的高性能还是值得的。更多感受还要深入使用以后才有感觉。

由于作者水平有限，如有错漏，还请指正。