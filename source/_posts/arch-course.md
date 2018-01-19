---
title: 浙江大学 计算机体系结构 课堂笔记
date: 2017-09-26 14:00:04
tags:
- 课程
---

仅包含部分提纲。

等死吧，老子不整理了。

<!-- more -->

### Flynn 分类法

按照指令流与数据流进行分类。

- SISD
- SIMD
- MISD
- MIMD

## Pipeline Hazard

- Structural hazards
- Data hazards
- Control hazards

## MIPS 五级流水线

- IF
  - 取指
  - 指令和 NPC (PC + 1) 写入下一级
- ID
  - 译码 读取寄存器堆
  - 读出 A B 寄存器与立即数传入下一级
- EXE
  - 执行
  - 进行算术运算或者计算跳转指令的有效地址
  - branch 指令同时计算条件
- MEM
  - 访存
  - 内存引用：读或者写内存
  - branch：写 PC
- WB
  - 写回
  - load 的内存或者运算结果写入运算目标寄存器

## Forwarding

- MEM -> EX
- EX -> EX
- WB -> MEM ?
- MEM -> MEM
- WB -> EX