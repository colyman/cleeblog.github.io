---
layout: article
title: "BJMM Algorithm：1+1=0 如何改进信息集解码"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
hidden: true
---

**tl;dr**: BJMM (Eurocrypt 2012) 将随机线性码解码的复杂度从 MMT 的 2^{0.05364n} 降至 2^{0.04934n}。核心创新是允许索引集合重叠，利用 GF(2) 中 1+1=0 的性质，同时拆分 1-位和 0-位，大幅增加解的表示数量。

<!-- more -->

## 1. Background: Information Set Decoding

随机线性码的解码问题是编码密码学和安全分析的核心。设 H in GF(2)^{(n-k)×n} 为一致校验矩阵，s = He 为错误向量 e 的伴随式，解码问题即从 s 恢复 e，其中 wt(e) = ω。

Prange (1962) 提出的 **Information Set Decoding (ISD)** 是最基础的通用算法。其核心思想是：对 H 的列进行随机置换后做高斯消元得到准系统形式 (Q | I_{n-k})，然后在前 k 位中搜索重量为 p 的向量来恢复完整错误向量。

后续优化路径：

- **Lee-Brickell (1988)**：利用 p < ω -> 2^{0.05752n}
- **Stern (1989)**：Meet-in-the-Middle (MITM) -> 2^{0.05564n}
- **Ball-collision** (Bernstein, Lange & Peters, CRYPTO 2011)：非精确匹配 -> 2^{0.05559n}
- **MMT** (May, Meurer & Thomae, Asiacrypt 2011)：表示技术 -> 2^{0.05364n}

## 2. Submatrix Matching Problem (SMP)

MMT 将 ISD 的搜索阶段归结为**子矩阵匹配问题（Submatrix Matching Problem, SMP）**：

> 给定随机矩阵 Q in GF(2)^{l×(k+l)} 和目标向量 s in GF(2)^l，求集合 I subseteq [1, k+l]，|I|=p，使得 sigma(Q_I) := sum_{i in I} q_i = s。

MMT 的核心思路是**表示**：将目标集合 I 写成 I = I_1 cup I_2，其中 |I_1| = |I_2| = p/2，I_1 cap I_2 =varnothing。每个真实解 I 对应 C(p, p/2) 种不同的 (I_1, I_2) 配对。

## 3. 关键洞察：1+1=0

MMT 的局限在于：I_1 和 I_2 必须**不相交**，这意味着 MMT 只能拆分 1-位：1 = 1+0 或 1 = 0+1。

**BJMM 的核心创新**：打破不相交约束，允许 |I_1 cap I_2| = epsilon > 0。此时解集为**对称差**：

    I = I_1 Delta I_2 = (I_1 cup I_2) \ (I_1 cap I_2)

这带来了关键突破：**0-位也可以被拆分！**

- 一个不在 I 中的 0-位可以同时出现在 I_1 和 I_2 中（即 (1,1)），由于 1+1=0 在 GF(2) 中，其对对称差的贡献仍为 0。
- 或者不出现（(0,0)），贡献也为 0。

> **表示数量对比**：
> - MMT：每个解有 C(p, p/2) 种表示
> - BJMM：每个解有 C(p, p/2) · C(k+l-p, epsilon) 种表示

由于 k+l-p >> p（码字中 0 的数量远大于 1 的数量），这个额外因子非常显著。

## 4. COLUMN MATCH: 三层算法

为了利用上述扩展表示，BJMM 提出了一个三层（three-level）的 divide-and-conquer 算法，结构类似于 Wagner 的广义生日悖论算法。

### 4.1 参数定义

    p_1 = p/2 + epsilon_1,    p_2 = p_1/2 + epsilon_2

表示数量为：

    R_1 = C(p, p/2) · C(k+l-p, epsilon_1),    R_2 = C(p_1, p_1/2) · C(k+l-p_1, epsilon_2)

### 4.2 MERGE-JOIN: 基础构建块

**MERGE-JOIN** 的功能是：给定两个列表 L_1, L_2，找出所有满足以下条件的配对 (x, y)：

1. wt(x + y) = p（重量约束）
2. (Q(x+y))[r] = t（前 r 位匹配目标 t）

通过排序 + 双指针高效实现，时间复杂度为 O(max{|L_1|, |L_2|, C})。

### 4.3 三层结构

算法由底向上：

- **Layer 3**：构造 4 对基列表，基于随机划分不相交拆分
- **Layer 2**：MERGE-JOIN 得到重量为 p_2 的向量
- **Layer 1**：MERGE-JOIN 得到重量为 p_1 的向量
- **Layer 0**：最终 MERGE-JOIN，匹配全部 l 位、重量 p，输出 SMP 的解

正确性由对称差操作的性质保证：对于 |I_1| = |I_2| = p_1 + epsilon_1，|I_1 cap I_2| = epsilon_1，有 |I_1 Delta I_2| = p。

## 5. 复杂度分析

各层列表大小为 S_2 = C(k+l, p_2) / 2^{r_2}，S_1 = C(k+l, p_1) / 2^{r_1}，其中 r_i ≈ log_2 R_i。

最优参数给出各复杂度系数：

| 算法 | 半距离解码 | 存储系数 |
|------|-----------|---------|
| Lee-Brickell | 2^{0.05752n} | — |
| Stern | 2^{0.05564n} | 2^{0.0135n} |
| Ball-collision | 2^{0.05559n} | 2^{0.0148n} |
| MMT | 2^{0.05364n} | 2^{0.0216n} |
| **BJMM** | 2^{0.04934n} | 2^{0.0286n} |

对于典型 McEliece 参数（相对距离 D=0.04，码率 R=0.7577），BJMM 的复杂度为 2^{0.0672n}，相较 MMT 的 2^{0.0760n} 有约 2^{0.0088n} 的指数改进。

## 6. 小结

BJMM 的核心贡献在于**扩展了表示技术**：通过允许索引集合重叠，巧妙利用了 GF(2) 中 1+1=0 的代数性质，将 0-位的组合自由度纳入表示空间。后续的 sieving-based ISD 算法进一步将复杂度压低，但 BJMM 的三路分解框架仍是理解现代 ISD 算法演进的必经之路。

## References

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in 2^{n/20}: How 1+1=0 Improves Information Set Decoding*. Eurocrypt 2012. ([ePrint 2012/026](https://eprint.iacr.org/2012/026))
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in O~(2^{0.054n})*. Asiacrypt 2011.
4. D. J. Bernstein, T. Lange, C. Peters. *Smaller Decoding Exponents: Ball-Collision Decoding*. Crypto 2011.
