---
layout: article
title: "BJMM Algorithm：1+1=0 如何改进信息集解码"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD]
hidden: true
---

**tl;dr**: BJMM (Eurocrypt 2012) 将随机线性码解码的复杂度从 MMT 的 2^{0.05364n} 降至 2^{0.04934n}。

<!-- more -->

## 1. Background

Information Set Decoding (ISD) 是解码随机线性码的基础算法。

## 2. Submatrix Matching Problem

SMP 是 ISD 搜索阶段的核心子问题。

## 3. BJMM 核心洞察

允许索引集合重叠，利用 F2 中 1+1=0 的性质。

## References

1. Becker, Joux, May, Meurer. Eurocrypt 2012.
