---
title: 语言服务器协议 - 规范阅读笔记
date: 2019-10-20 21:59:34
tags:
- Language Server Protocol
- 开发工具
---

## 语言服务器协议 - Language Server Protocol

记一下阅读笔记，因为想了解下这东西的实现原理，否则用不明白。

[overview](https://microsoft.github.io/language-server-protocol/overview)

[specification](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/)

<!-- more -->

这里的 **Overview** 很好，看完之后就有个大概了。但我仍然有几个问题，比如工作区管理的概念是怎样的？双方如何同步状态？大量的源代码由何种格式被维护，由谁维护？

## 基本协议

基本协议格式就是去掉请求行的 HTTP 协议。要求至少有一条 Header 内容，一定要有 `Content-Length` 不然和 HTTP 一样无法得知内容长度。

请求体就是 `JSON-RPC` 文本，版本固定 `2.0`。

协议都是文本的，很方便感兴趣的人直接抓取。看起来这些请求的交互都是双向的。总共有 `Request` `Response` 和 `Notification` 三种通信对象。前两者一问一答，必须成对出现，通知则不需要被回复。且 client 和 server 谁都可以发出请求，谁也都可以发出通知。

- 服务器的生命周期由客户端控制，客户端例如 `VSCode` 和 `Emacs` 控制进程级别的服务器启动与停止。

- 协议中感知的对象和概念

  - 文本文档
  
    - 位置
	
	- 范围
	
	- 文本段
	
	- etc...
  
  - 工作区

## 基本设计

由于不同的语言区别巨大且难以抽象，而且抽象语法树的构建本来就该是语言工具要做的事情，如果协议层面有这个概念就需要客户端完成相关操作了。所以双方交互的文本就像文本编辑器一样直接，纯文本，没有经过词法和语法的任何处理。

为了不让服务端过于繁重，经常处理整个文件的变更，协议层面可以传递增删改查，客户端传递变更过去就可以了。

用户在客户端执行诸如“跳转到定义”这种命令的时候，会以 `Request` and `Response` 来交互，因为这个请求必须被及时处理。而添加文件，服务端输出诊断消息等场景往往使用 `Notification`，这语义上更像异步通信，给了服务端充足的时间和自由进行处理。尽管这样，不要求请求的顺序和响应的顺序一一对应，服务端保证重排之后的语义没有变化即可。


一个客户端往往同时处理多个语言，可以打开不同的语言对应的服务端。

服务端启动后处理的第一个请求应该是初始化请求，它执行完之前不应该响应其他请求。

## 工作区处理

一个服务端进程可以处理单个文件，也可以感知单根或者多根的工作区。即 一个 `Workspace` 可以包含多个 `Folder`。

工作区包含的文件夹在初始化时声明，后续的过程中也可以更改。

存疑：一组客户端/服务端实例只维护一个工作区？没有看到多个工作区相关的 spec。

## 持久化

这部分我还没看到答案。随着客户端关闭，服务端随之关闭，那么工作区巨大的情况下，语言服务器建立好的信息要不要保存？有没有规范？如果有，服务端不同进程生命周期之间如何保证信息正确？

## 杂项

一些我没那么感兴趣的内容。

### `Capabilities`

毕竟协议内容太多了，有一说一，确实，不是随便做一个服务器就都能实现完成的。比起简单粗暴的看情况来，定义了功能集作为“能力”标注，由服务器选择性地分组支持。

同时客户端也支持“能力”标注。
