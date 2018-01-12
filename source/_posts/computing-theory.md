---
title: 浙江大学 计算理论 提纲
date: 2017-10-13 10:08:50
tags:
- 课程
- 计算理论
---

天塌了（

<!-- more -->

| 章节列表     |
| -------- |
| 集合，关系和语言 |
| 有限自动机    |
| 上下文无关语言  |
| 图灵机      |
| 不确定性     |



| 语言      | 自动机           |
| ------- | ------------- |
| 正则语言    | 确定性/非确定性有限自动机 |
| 上下文无关语言 | 下推自动机         |
| 递归可枚举语言 | 图灵机           |

## TODO list

- [ ] 各种语言的 closure property
- [ ] DFA <=> NFA
- [ ] 泵定理 * 2
- [ ] primitive recursive function

## 14 - 15年考卷

### 判断题

#### (a)

False

#### (b)

False

$\{a^ib^jc^k|i,j,k\in\mathbb{N}~\mathrm{and}~i+j\not\equiv k~(\mod3)\}$

#### (c)

True

regular与non-regular并运算

## 复习PPT考点

- 正则语言
  - DFA / NFA => regex
  - regex => DFA / NFA
  - 判断一个语言是否正则
    - 是：DFA / NFA / closure property
    - 否：closure property / pumping theorem
- 上下文无关语言
  - context-free lang => grammar
  - grammar => PDA
  - lang => PDA
  - 判断一个语言是否上下文无关
- 递归可枚举语言
  - 设计图灵机计算一个函数 / 判定 半判定语言
  - TM => function
  - 判断函数是否为 primitive recursive function
- 不确定性
  - 判断语言递归可枚举 / 不递归
- P与NP
  - 判断语言属于P问题还是NP问题
- NP完全
  - show a given lang be NP-complete by reduction