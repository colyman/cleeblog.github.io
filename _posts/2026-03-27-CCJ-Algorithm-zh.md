---
layout: post
title: "CCJ Algorithm: 线性码中找最小重量码字的迭代算法"
date: 2026-03-27
tags: [Code-Based-Cryptography, McEliece, CCJ-Algorithm, Markov-Chain]
author: tanglee
---

## tl;dr

最小重量码字问题（MWCP）是编码密码学的核心难题：给定一个线性码，如何高效找到重量最小的非零码字？Canteaut、Chabaud、Flandre 和 Levy 在 1998 年的这篇论文中提出了一个精妙的改进算法，通过**相邻信息集的迭代策略**替代了 Stern 算法中代价高昂的重复高斯消元，将 McEliece 密码系统的攻击效率提升了约 128 倍。

<!--more-->

## 背景: 最小重量码字问题

设 $\mathcal{C}$ 是一个 $[n, k]$-线性的二元码，其生成矩阵为 $G \in \mathbb{F}_2^{k \times n}$。我们希望找到非零码字中重量（汉明重量）最小的那个：

$$\min_{c \in \mathcal{C} \setminus \{0\}} \mathrm{wt}(c)$$

这个问题在 1978 年被 Berlekamp、McEliece 和 van Tilborg 证明是 **NP 完全** 的。然而，NP 完全并不意味着没有比暴力搜索更快的算法——暴力搜索复杂度为 $O(n \cdot 2^k)$，而 CCJ 算法的复杂度约为 $O(2^{0.1n})$（对于 $[n, n/2]$ 类型的码）。

**为什么重要？** McEliece 公钥密码系统 [McEliece 1978] 的安全性建立在：给定公钥生成矩阵 $G'$（无明显结构），攻击者无法高效地解码或找到最小重量码字。任何改进 MWCP 算法的工作都直接影响这类密码系统的安全参数选取。

## Stern 算法 (1989)

在 CCJ 之前，Stern（1989）提出了最有效的攻击算法，其核心步骤如下：

1. **随机选取信息集** $I \subseteq \{1, \ldots, n\}$，$|I| = k$
2. **高斯消元**：将 $G$ 化为系统形式 $[I_k \mid A]$（$O(n^3)$ 代价）
3. **随机分裂**：将 $I$ 随机分为 $I_1$（$p$ 个元素）和 $I_2$（$k-p$ 个元素）
4. **随机列选择**：随机选取冗余位集合 $J \subseteq \{1,\ldots,n\}\setminus I$，$|J| = \ell$
5. **哈希表搜索**：将所有 $I_1$ 行（$p$-子集）的 $J$-投影存入哈希表，再搜索 $I_2$ 行（$p$-子集）的碰撞

**问题所在**：每轮迭代都要重新做一次高斯消元（$O(n^3)$），这成为算法的主要瓶颈。

## CCJ 核心思想: 相邻信息集

CCJ 算法的关键洞察是：**不必要每次迭代都做完整的高斯消元**。

定义两个信息集 $I$ 和 $I'$ 是**相邻的**（close），若 $|I \triangle I'| = 2$，即它们只差一个元素。那么存在 $i \in I$ 和 $j \notin I$，使得 $I' = (I \setminus \{i\}) \cup \{j\}$。

**核心引理**：从系统形式 $[I \mid A]$ 出发，更新到相邻信息集 $I'$ 的系统形式 $[I' \mid A']$，只需一次**局部 pivoting 操作**（$O(n)$），无需重新高斯消元。

因此，CCJ 算法从一个随机信息集出发，通过一系列相邻信息集进行迭代，每次迭代的成本从 $O(n^3)$ 降到了 $O(n)$。

## 算法详解

设目标码字为 $w$，$S = \mathrm{supp}(w)$ 为其支撑集（重量为 $|S| = d$）。

### 初始化

1. 随机选取信息集 $I$
2. 高斯消元得到系统形式 $[I_k \mid A]$

### 每次迭代

**Phase 1：哈希搜索**

令 $I$ 被随机分裂为 $I_1$（$p$ 个）和 $I_2$（$k-p$ 个）。

- 对 $I_1$ 的每一组 $p$-子集行，计算其在 $J$ 上的投影 $\pi_J(c_1)$，存入哈希表（规模约 $2^p$ 项）
- 对 $I_2$ 的每一组 $p$-子集行，计算 $\pi_J(c_2)$，若存在碰撞（$\pi_J(c_1) = \pi_J(c_2)$），检查 $c_1 \oplus c_2$ 的全重是否等于 $d$

**条件**：若 $|S \cap I| = p$ 且 $S \setminus I \subseteq J$，则碰撞对应的 $c_1 \oplus c_2$ 即为目标码字。

**Phase 2：信息集更新**

随机选取 $i \in I$ 和 $j \notin I$，令 $I \leftarrow (I \setminus \{i\}) \cup \{j\}$，通过 pivoting 更新 $A$（$O(n)$ 复杂度）。

### 马尔可夫链建模

CCJ 巧妙地将算法建模为**齐次马尔可夫链**：

- **状态**：当前信息集中包含的目标码字非零位的数量（记为 $w_i = |S \cap I|$）
- **状态空间**：$\{1, \ldots, d\} \cup \{p\} \times \{\sigma, \tau\}$（其中 $\sigma, \tau$ 区分成功前的两种边界情况）
- **转移**：改变 $I$ 中的一个元素使 $w_i$ 最多变化 1

论文证明了：
1. 这是一个**吸收马尔可夫链**，唯一吸收态对应成功状态
2. 无论从哪个状态出发，算法最终收敛（概率为 1）
3. 期望迭代次数由**基本矩阵**（fundamental matrix）给出

## 复杂度分析

论文给出了精确的**参数优化公式**。对于随机 $[n, n/2]$-二元码（扩展率 $\rho = 1/2$）：

### 解码（到纠错能力 $t$）

$$\text{WF} \approx n \cdot 2^{0.1 \cdot n}$$

其中 $n$ 为码长，$0.1$ 来源于熵函数在 $\rho = 1/2$ 处的斜率。

### 找最小重量码字

$$\text{WF} \approx n \cdot 2^{0.11 \cdot n}$$

**实验数据**（论文 Table II）：

| 码参数 | 最优 $p$ | 最优 $\ell$ | 期望迭代次数 | Work Factor |
|--------|---------|------------|-------------|-------------|
| $[512, 256, 57]$ | 8 | 14 | $2^{41}$ | $2^{67}$ |
| $[1024, 512, 57]$ | 9 | 15 | $2^{68}$ | $2^{95}$ |

## 对 McEliece 密码系统的攻击

CCJ 算法构成了对 McEliece/Niederreiter 公钥密码系统已知最有效的攻击。

| 算法 | $[1024, 524]$ Goppa 码 WF |
|------|--------------------------|
| Lee-Brickell (1988) | $2^{70}$ |
| **CCJ (1998)** | $2^{64}$ |
| 提升倍数 | **~128×** |

对于高码率场景（如 $[1024, 1019]$），CCJ 的 work factor 仅约 $2^{37}$，这意味着降低公钥尺寸（缩短码长）在安全性上并不可行。

## 应用: BCH 码最小距离

论文还将 CCJ 算法应用于长度 $511 = 2^9 - 1$ 的 primitive narrow-sense BCH 码，确证了多条 BCH 码的真实最小距离（此前只知道 BCH 界）。这是该算法在编码理论中的另一重要应用。

## 总结

CCJ 算法的贡献在于三点：

1. **算法改进**：用"相邻信息集"的迭代策略替代重复高斯消元，大幅降低每轮迭代成本
2. **理论分析**：通过马尔可夫链精确刻画算法收敛性，给出可计算的 work factor 近似公式
3. **实践影响**：将 McEliece 系统的主流攻击效率提升 2 个数量级，直接影响安全参数选取

这篇论文是编码密码学和密码分析的经典文献，也是理解现代码基密码系统安全边界的重要参考。

---

**参考文献**：
- Canteaut, A., & Chabaud, F. (1998). A New Algorithm for Finding Minimum-Weight Words in a Linear Code: Application to McEliece's Cryptosystem and to Narrow-Sense BCH Codes of Length 511. *IEEE Transactions on Information Theory*, 44(1), 367–378.
- Stern, J. (1989). A Method for Finding Codewords of Small Weight. *Coding Theory and Applications*, 106–113.
- McEliece, R. J. (1978). A Public-Key Cryptosystem Based on Algebraic Coding Theory. *DSN Progress Report*, 114–116.
