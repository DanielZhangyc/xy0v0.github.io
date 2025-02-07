---
date: 2025-02-08T00:10:50+08:00
tags:
  - OI
  - 动态规划
title: OI | ABC 391 E 题解整理
slug: "1738944650"
share: true
searchHidden: false
disableShare: true
math: true
mermaid: false
canonicalURL: ""
keywords: 
description: 文章讨论了如何通过分组操作将一个长度为 \(3^N\) 的二进制字符串 \(A\) 最终转化为一个长度为1的字符串 \(A'1\)，并找出改变最终结果所需的最小变更数。每次操作将字符串分为三元组，取每组的多数值生成新字符串。分析表明，生成0的三元组有0、1、2、4个0，生成1的有1、2、3个1。当三元组相同时，需改变2个元素，否则只需改变1个。解决方案采用递归建树，视三元组为叶子节点，定义代价 \(Cost\) 作为改变最终结果所需的最小变更数，通过自底向上计算。叶子节点的 \(Cost\) 为2或1，祖先节点计算时考虑多种变更策略，选择代价最小的路径。通过动态规划和递归，能有效计算出最小变更数。
series: 系列
lastmod: 
lang: cn
cover:
    image: ""
author: 
dir: posts
---
## 题面

>For a binary string $B = B_1 B_2 \dots B_{3^n}$ of length $3^n$ ($n \geq 1$), we define an operation to obtain a binary string $C = C_1 C_2 \dots C_{3^{n-1}}$ of length $3^{n-1}$ as follows:
>
>-   Partition the elements of $B$ into groups of $3$ and take the majority value from each group. That is, for $i=1,2,\dots,3^{n-1}$, let $C_i$ be the value that appears most frequently among $B_{3i-2}$, $B_{3i-1}$, and $B_{3i}$.
>
>You are given a binary string $A = A_1 A_2 \dots A_{3^N}$ of length $3^N$. Let $A' = A'_1$ be the length-$1$ string obtained by applying the above operation $N$ times to $A$.
>
  Determine the minimum number of elements of $A$ that must be changed (from $0$ to $1$ or from $1$ to $0$) in order to change the value of $A'_1$.

简单来说，给定01字符串 $A$ ，长度为 $3^n$ ，每次操作将字符串分割为三个为一组，取出现最多的数替代这三个数字，要使最终得到的字符串改变结果 $(0 \rightarrow 1 \ or \ 1 \rightarrow 0)$  ，最少需要改变几个数字

## 题目分析

首先进行分析 ，不难发现只有 $0,1,2,4$ 经过操作后得到的是 $0$ ，其余情况得到的均为 $1$ 。要将三个一组的数经过操作得到的结果变为另一种，需要改变1/2个数字，其中改变2个数字的有 $000, 111$ 其余情况均只需改变一个数字。我们可以定义要改变的数字数量为代价，考虑建图。

## 解题思路

可以考虑递归建树，可将字符串 $A$ 视为三叉树的叶子结点，对每个节可定义 $Cost$ ，代表改变为另一个数所需的代价。首先计算叶子节点的 $Cost$ ，非叶子节点可由叶子节点递推得到。

举例：000110001

叶子结点计算得：
$$ 
C_{1}=2, C_{2} = 1 , C_{3} = 1 
$$
其祖先节点计算原始多数值为 $010$ ，计算其 $C_1$ 时可以选择改变第一位，也可以选择改变第三位，但是显然改变第三位更优。

对于如何简易计算 $Cost$ ，我们不难发现当三数字相同时候需改变其中两个数字，改变两个和最小的即可。当两数字相同时只需改变两相同数字中较小的那个。