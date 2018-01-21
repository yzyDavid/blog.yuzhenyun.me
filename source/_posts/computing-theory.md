---
title: 浙江大学 计算理论 提纲
date: 2017-10-13 10:08:50
tags:
- 课程
- 计算理论
---

天塌了（



| 章节列表     |
| -------- |
| 集合，关系和语言 |
| 有限自动机    |
| 上下文无关语言  |
| 图灵机      |
| 不可判定性    |

| 语言      | 自动机           |
| ------- | ------------- |
| 正则语言    | 确定性/非确定性有限自动机 |
| 上下文无关语言 | 下推自动机         |
| 递归可枚举语言 | 图灵机           |

## TODO list

- 各种语言的 closure property
- 泵定理 * 2
- primitive recursive function
- automata encoding
- simple TMs

<!-- more -->

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

#### (d)

False

正则的子集不一定正则

#### (e)

$\{xcy|x,y\in\{a,b\}^*~\mathbb{and}~|x|\le|y|\le2|x|\}$ is context-free

泵定理

#### (f)

#### (g)

#### (h)

#### (i)

#### (j)

#### (k)

#### (l)

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
- 不可判定性
  - 判断语言递归可枚举 / 不递归
- P与NP
  - 判断语言属于P问题还是NP问题
- NP完全
  - show a given lang be NP-complete by reduction

## 知识点

### 下推自动机

六元组 $M=(K,\Sigma,\Gamma,\Delta,s,F)$

- $K$ 状态集合
- $\Sigma$ 输入字母表
- $\Gamma$ 栈字母表
- $s\in K$ 初始状态
- $F\subseteq K$ 终结状态集
- $\Delta$ 转移函数

$\Delta=\{((p,\alpha,\beta),(q,\gamma))\}$

- $p$ PDA 当前的状态
- $\alpha$ 读入的字母
- $\beta$ 栈顶的元素
- $q$ 要转移到的状态
- $\gamma$ 将栈顶的元素修改到的元素

接受条件：

- 处理完输入，栈为空
- PDA 到达终结状态（之一）

### 图灵机

五元组 $M=(K,\Sigma,\delta,s,H)$

- $K$ 状态集合
- $\Sigma$ 输入字母表
  - $\sqcup$ 为空符号
  - $\rhd$ 为纸带最左端
- $s\in K$ 初始状态
- $H\subseteq K$ 停机状态集
- $\delta$ 转移函数

$K=\{(q,w\underline au)\}$

这里与张海的复习笔记有出入，后者记这个集合是 $\delta$。

### Context Free Grammar 上下文无关语法

$G=(V,\Sigma,R,S),R=\{A\to u\}$

其中：

- $V$ 为字母表
- $\Sigma\subseteq V$ 为终结符号的集合
- $S\in V-\Sigma$ 为起始符号
- $R$ 为规则，是$(V-\Sigma)\times V^*$ 的子集

#### Chomsky Normal Form

TODO

#### CFL to CFG

手动构造。

常见例子就是回文，两种字母个数相等，字母个数有等式或者不等式限制等等。

#### CFL to PDA

也只能手动构造，同上。

#### CFG to PDA

$G=(V,\Sigma,R,S)$

$M=(K,\Sigma,\Gamma,\Delta,s,F)$

给出文法 $G$，则 PDA 的参数可定义为：

- $K=\{p,q\}$
- $\Gamma=V$
- $s=p$
- $F=\{q\}$
- $\Delta$ 包括以下三种变换：
  - $((p,e,e),(q,S))$ 接受
  - $(q,e,A),(q,x)$ 生成, for each rule $A\to x\in R$
  - $((q,a,a),(q,e)), \forall a\in\Sigma$ 匹配

### 对于正则语言的泵定理

Let $L$ be a regular language, then there exists an integer $n\ge1$ that any string $w\in L$ with $|w|\ge n$ can be written as $w=xyz$ such that:

- $y\ne e$
- $|xy|\le n$
- $\forall i>0,xy^iz\in L$

~~想不明白了，先睡觉。~~

先背下来再说。

### 对于上下文无关语言的泵定理

#### 引理

高度为 $h$ 的任何 $G$ 的parse tree 长度不超过 $\phi(G)^h$

#### 定理

Let $G=(V,\Sigma,R,S)$ be a CFG, then any string $w\in L(G)$ of length greater than $n\ge \phi(G)^{|V-\Sigma|}$ can be written as $w=uvxyz$ such that:

- $vy\ne e$
- $|vxy|\le n$
- $\forall i \ge 0,uv^ixy^iz\in L(G)$

### NFA to DFA

子集构造法

DFA 的每个状态对应 NFA 的一个状态集合。

输入：NFA $N$

输出：DFA $D$

构造转换表 $Dtran$

$D$ 的状态集合 $Dstates$

$N$ 的单个状态记为 $s$，$N$ 的一个状态集记为 $T$。

$\epsilon-closure(s)$

$\epsilon-closure(T)=U_{s\in T}\epsilon-closure(s)$

$move(T,a)$ 从 $T$ 的某个状态 $s$ 出发，通过标号为 $a$ 的转换能到达的 NFA 的状态集合。

> 开始时，$\epsilon-closure(s_0)$ 是 $Dstates$ 中的唯一状态且未加标记。
>
> while $Dstates$ 中有一个未标记状态 $T$
>
> ​	标记 $T$
>
> ​	foreach 输入符号 $a$
>
> ​		$U=\epsilon-closure(move(T,a))$
>
> ​		if $U\not\in Dstates$
>
> ​			将 $U$ 加入 $Dstates$ 且不加标记
>
> ​		$Dtran[T,a]=U$

### 判断语言是否为正则语言

正则语言对于交、并、补、差，连接，克林闭包封闭。

- F - 正则语言的子集都是正则语言
- F - 正则语言都有正则的真子集（空集没有）
- T - 正则语言 $L~x,y\in L,\{xy|x\in L,y\not\in L\}$ 是正则语言（补集与连接）
- F - $\{w|w=w^R\}$ 是正则语言
- T - $\{w|w\in L ~\text{and}~ w^R\in L\}$ （$L$ 是正则语言，$L=L\cap L^R$）???
- F - If $C$ is any set of regular languages, then $\cup C$ is a regular language. ???
- T - $\{xyx^R|x,y=\Sigma^*\}$ 是正则语言（$x$ 可以为空，$y=\Sigma^*$ ）

### 判断语言是否为上下文无关

上下文无关语言对于并，连接与克林闭包封闭，对交、补、差**不**封闭。

正则语言与上下文无关语言的交集还是上下文无关语言。

### 判断语言是否为递归可枚举

递归可枚举语言对于交，并，连接和克林闭包运算封闭。对补与差运算不封闭。

### 不可判定性

$L(M)$ : 图灵机 $M$ 半判定的语言。 

Tiling 问题是不可解的。

图灵机有可数无限多个。

#### 邱奇-图灵论题

#### 算法

#### 判定

#### 半判定

#### 不可判定

#### 函数

#### 递归语言

图灵机可以停机并给出判定的语言。

若语言 $L$ 与 $\bar{L}$ 均为递归可枚举语言（即正反都能停机判定），则 $L$ 为递归语言。

对于补运算封闭。

#### 递归可枚举语言

图灵机对于接受的字符串可以停机，对于不接受的字符串可能不停机的语言。

递归可枚举语言即图灵可枚举语言。

#### 通用图灵机

$U(M,w)=M(w)$

### Rice 定理

设 P 是任何图灵机的语言的非平凡属性。确定某一图灵机的语言是否具有属性 P 这一问题是不可判定的。

If $C$ is a proper and non-empty subset of recursively enumerable languages, then it is undecidable for a Turing Machine $M$, whether $L(M)\in C$

> 所以几乎所有关于“图灵机是否在此种输入上停机”的问题都是不可判定的。

e.g:

- $L(M)=\emptyset$
- $L(M)$ is finite
- $L(M)=\Sigma^*,e\in L(M)$

