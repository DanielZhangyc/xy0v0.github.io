---
date: 2025-03-26T23:12:35+08:00
tags: 
title: 学习笔记 | BGE-m3中的检索方式
slug: "1743001955"
share: true
searchHidden: false
disableShare: true
math: true
mermaid: false
canonicalURL: ""
description: ""
series: 系列
lastmod: 
lang: cn
cover:
    image: ""
author: 
dir: posts
---

## **密集检索（Dense Retrieval）**

使用 CLS 标记的输出嵌入计算相似度，公式为：

$$
  s_{\text{dense}} = \langle e_q, e_p \rangle
$$

其中 $e_q$ 和 $e_p$ 分别表示查询和表情包描述的嵌入向量。这种基于语义的检索方式能够捕捉用户问题的深层含义。

##  **稀疏检索（Sparse Retrieval）**

- 通过计算词项权重实现检索，公式为：
 $$ 
  s_{\text{lex}} = \sum_{t \in q \cap p} (w_{qt} \cdot w_{pt})
$$
  其中 $w_{qt}$ 和 $w_{pt}$ 分别表示查询和表情包描述中词项 \(t\) 的权重。这种检索方式适合处理关键词匹配问题。

## **多向量检索（Multi-Vector Retrieval）**

- 利用细粒度的 token 级别交互计算相关性，公式为：
  
$$
 s_{\text{mul}} = \frac{1}{N} \sum_{i=1}^N \max_{j=1}^M (e_{qi} \cdot e_{pj})
$$
  
  其中 $e_{qi}$ 和 $e_{pj}$ 分别表示查询和表情包描述中第 $i$ 和第 $j$ 个 token 的嵌入向量。这种检索方式能够捕捉更细粒度的语义信息。

