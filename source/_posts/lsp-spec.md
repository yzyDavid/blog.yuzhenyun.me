---
title: 语言服务器协议 - 规范阅读笔记
date: 2019-10-20 21:59:34
tags:
---

## 语言服务器协议 - Language Server Protocol

记一下阅读笔记，因为想了解下这东西的实现原理，否则用不明白。

[specification](https://microsoft.github.io/language-server-protocol/specifications/specification-3-14/)

<!-- more -->

## 基本协议

基本协议就是去掉请求行的 HTTP 协议。要求至少有一条 Header 内容，一定要有 `Content-Length` 不然和 HTTP 一样无法得知内容长度。

请求体就是 `JSON-RPC` 文本，版本固定 `2.0`。

## 待续
