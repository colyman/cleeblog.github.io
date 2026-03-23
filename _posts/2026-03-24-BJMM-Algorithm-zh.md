---
layout: article
title: "BJMM Algorithm：1+1=0 如何改进信息集解码"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
hidden: true
---

**tl;dr**: BJMM (Eurocrypt 2012) 将随机线性码解码的复杂度从 MMT 的 $2^{0.05364n}$ 降至 $2^{0.04934n}$。核心创新是允许索引集合重叠，利用 $\mathbb{F}_2$ 中 $1+1=0$ 的性质，同时拆分 1-位和 0-位，大幅增加解的表示数量。

<!-- more -->

## 1. Background: Information Set Decoding

随机线性码的解码问题是编码密码学和安全分析的核心。设 $H \in \mathbb{F}_2^{(n-k)\times n}$ 为一致校验矩阵，$s = He$ 为错误向量 $e$ 的伴随式，解码问题即从 $s$ 恢复 $e$，其中 $\mathrm{wt}(e) = \omega$。

Prange (1962) 提出的 **Information Set Decoding (ISD)** 是最基础的通用算法。其核心思想是：对 $H$ 的列进行随机置换后做高斯消元得到准系统形式

$$
\tilde{H} = (Q \mid I_{n-k}),
$$

其中 $Q \in \mathbb{F}_2^{(n-k)\times k}$。设错误向量 $\tilde{e}$ 的前 $k$ 位中恰有 $p$ 个 1，后 $n-k$ 位中恰有 $\omega-p$ 个 1，则搜索 $\tilde{e}_1 \in \mathbb{F}_2^k$（满足 $\mathrm{wt}(\tilde{e}_1) = p$）即可恢复完整解。

后续优化路径：

- **Lee-Brickell (1988)**：$p < \omega$ 的优化 → $2^{0.05752n}$
- **Stern (1989)**：Meet-in-the-Middle (MITM) -> $2^{0.05564n}$
- **Ball-collision** (Bernstein, Lange & Peters, CRYPTO 2011)：非精确匹配 -> $2^{0.05559n}$
- **MMT** (May, Meurer & Thomae, Asiacrypt 2011)：表示技术 -> $2^{0.05364n}$

## 2. Submatrix Matching Problem (SMP)

MMT 将 ISD 的搜索阶段归结为**子矩阵匹配问题（Submatrix Matching Problem, SMP）**：

> 给定随机矩阵 $Q \in \mathbb{F}_2^{l \times (k+l)}$ 和目标向量 $s \in \mathbb{F}_2^l$，求集合 $I \subseteq [1, k+l]$，$|I|=p$，使得
>
> $$\sigma(Q_I) := \sum_{i \in I} q_i = s,$$
>
> 其中 $q_i$ 为 $Q$ 的第 $i$ 列。

MMT 的核心思路是**表示**：将目标集合 $I$ 写成 $I = I_1 \cup I_2$，其中 $|I_1| = |I_2| = p/2$，$I_1 \cap I_2 = \varnothing$。每个真实解 $I$ 对应 $\binom{p}{p/2}$ 种不同的 $(I_1, I_2)$ 配对，从而列表规模可以缩小 $\binom{p}{p/2}$ 倍。

## 3. 关键洞察：$1+1=0$

MMT 的局限在于：$I_1$ 和 $I_2$ 必须**不相交**，这意味着 MMT 只能拆分 1-位：

$$
1 = 1+0 \quad \text{或} \quad 1 = 0+1.
$$

**BJMM 的核心创新**：打破不相交约束，允许 $|I_1 \cap I_2| = \varepsilon > 0$。此时解集为**对称差**：

$$
I = I_1 \,\Delta\, I_2 = (I_1 \cup I_2) \setminus (I_1 \cap I_2).
$$

这带来了关键突破：**0-位也可以被拆分！**

- 一个不在 $I$ 中的 0-位可以同时出现在 $I_1$ 和 $I_2$ 中（即 $(1,1)$），由于 $1+1=0$ 在 $\mathbb{F}_2$ 中，其对对称差的贡献仍为 0。
- 或者不出现（$(0,0)$），贡献也为 0。

因此，0-位的分配方式提供了额外的组合自由度。

> **表示数量对比**：
> - MMT：每个解有 $\displaystyle \binom{p}{p/2}$ 种表示
> - BJMM：每个解有 $\displaystyle \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon}$ 种表示（增加因子 $\binom{k+l-p}{\varepsilon}$）

由于 $k+l-p \gg p$（码字中 0 的数量远大于 1 的数量），这个额外因子非常显著！

## 4. COLUMN MATCH: 三层算法

为了利用上述扩展表示，BJMM 提出了一个三层（three-level）的 divide-and-conquer 算法，结构类似于 Wagner 的广义生日悖论算法。

### 4.1 参数定义

$$
p_1 = \frac{p}{2} + \varepsilon_1, \quad
p_2 = \frac{p_1}{2} + \varepsilon_2.
$$

表示数量为：

$$
R_1 = \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon_1}, \quad
R_2 = \binom{p_1}{p_1/2} \cdot \binom{k+l-p_1}{\varepsilon_2}.
$$

### 4.2 MERGE-JOIN: 基础构建块

**MERGE-JOIN** 的功能是：给定两个列表 $L_1, L_2$，找出所有满足以下条件的配对 $(x, y) \in L_1 \times L_2$：

1. $\mathrm{wt}(x + y) = p$（重量约束）
2. $(Q(x+y))[r] = t$（前 $r$ 位匹配目标 $t$）

通过排序 + 双指针高效实现，时间复杂度为 $O(\max\{|L_1|, |L_2|, C\})$，其中 $C \approx |L_1||L_2|/2^r$。

### 4.3 三层结构

算法结构如下（由底向上）：

- **Layer 3**：构造 4 对基列表 $B_{i,1}, B_{i,2}$（$i=1..4$），每对基于随机划分 $(P_1, P_2)$ 不相交拆分
- **Layer 2**：$L^{(2)}_i = \text{MERGE-JOIN}(B_{i,1}, B_{i,2}, r_2, p_2, t^{(2)}_i)$，得到重量为 $p_2$ 的向量
- **Layer 1**：分别对 $(L^{(2)}_1, L^{(2)}_2)$ 和 $(L^{(2)}_3, L^{(2)}_4)$ 做 MERGE-JOIN，得到 $L^{(1)}_1, L^{(1)}_2$（重量为 $p_1$）
- **Layer 0**：对 $(L^{(1)}_1, L^{(1)}_2)$ 做最终 MERGE-JOIN，匹配全部 $l$ 位、重量 $p$，输出 SMP 的解

正确性由对称差操作的性质保证：对于 $|I_1| = |I_2| = p_1 + \varepsilon_1$，$|I_1 \cap I_2| = \varepsilon_1$，有 $|I_1 \Delta I_2| = p$。

## 5. 复杂度分析

### 5.1 各层规模

设 $S_3 = \binom{(k+l)/2}{p_2/2}$ 为基列表大小。假设部分和均匀分布，则各层列表大小为：

$$
S_2 = \binom{k+l}{p_2} \cdot 2^{-r_2}, \quad S_1 = \binom{k+l}{p_1} \cdot 2^{-r_1},
$$

其中 $r_i \approx \log_2 R_i$ 是第 $i$ 层约束的坐标数。

### 5.2 总体复杂度

整个 ISD 算法的期望运行时间为 $T_{\text{ISD}} = P^{-1} \cdot T(p, l; \varepsilon_1, \varepsilon_2)$，其中迭代概率 $P = \frac{\binom{k+l}{p}\binom{n-k-l}{\omega-p}}{\binom{n}{\omega}}$。

最优参数给出各复杂度系数：

| 算法 | 半距离解码 | 存储系数 |
|------|-----------|---------|
| Lee-Brickell | $2^{0.05752n}$ | — |
| Stern | $2^{0.05564n}$ | $2^{0.0135n}$ |
| Ball-collision | $2^{0.05559n}$ | $2^{0.0148n}$ |
| MMT | $2^{0.05364n}$ | $2^{0.0216n}$ |
| **BJMM** | $2^{0.04934n}$ | $2^{0.0286n}$ |

对于典型 McEliece 参数（相对距离 $D=0.04$，码率 $R=0.7577$），BJMM 的复杂度为 $2^{0.0672n}$，相较 MMT 的 $2^{0.0760n}$ 有约 $2^{0.0088n}$ 的指数改进。

## 6. 小结

BJMM 的核心贡献在于**扩展了表示技术**：通过允许索引集合重叠，巧妙利用了 $\mathbb{F}_2$ 中 $1+1=0$ 的代数性质，将 0-位的组合自由度纳入表示空间。这一洞察看似简单，但实际效果显著——将解码复杂度从 $2^{0.05364n}$ 降至 $2^{0.04934n}$。

后续工作（如 sieving-based ISD）进一步将复杂度压低，但 BJMM 的三路分解框架和"1+1=0"的思想，仍是理解现代 ISD 算法演进的必经之路。

## References

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in $2^{n/20}$: How $1+1=0$ Improves Information Set Decoding*. Eurocrypt 2012. ([ePrint 2012/026](https://eprint.iacr.org/2012/026))
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in $\tilde{O}(2^{0.054n})$*. Asiacrypt 2011.
4. D. J. Bernstein, T. Lange, C. Peters. *Smaller Decoding Exponents: Ball-Collision Decoding*. Crypto 2011.
