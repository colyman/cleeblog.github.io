---
layout: article
title: "BJMM Algorithm：1+1=0 如何改进信息集解码"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
hidden: true
---

> **tl;dr**: BJMM (Eurocrypt 2012) 将随机线性码解码的复杂度从 MMT 的 $2^{0.05364n}$ 降至 $2^{0.04934n}$。核心创新是**允许索引集合重叠**（$|I_1 \cap I_2| = \varepsilon$），从而利用 $\mathbb{F}_2$ 中 $1+1=0$ 的性质，同时拆分 1-位和 0-位，大幅增加解的表示数量（representations），进而降低算法的时间复杂度。

<!-- more -->

## 1. Background: Information Set Decoding

随机线性码的解码问题（Syndrome Decoding）是编码密码学和安全分析的核心。设 $H \in \mathbb{F}_2^{(n-k)\times n}$ 为一致校验矩阵，$s = He$ 为错误向量 $e$ 的伴随式，解码问题即从 $s$ 恢复 $e$，其中 $\mathrm{wt}(e) = \omega$。

Prange (1962) 提出的 **Information Set Decoding (ISD)** 是最基础的通用算法。其核心思想是：对 $H$ 的列进行随机置换后做高斯消元得到准系统形式

$$
\tilde{H} = (Q \mid I_{n-k}),
$$

其中 $Q \in \mathbb{F}_2^{(n-k)\times k}$。设错误向量 $\tilde{e}$ 的前 $k$ 位中恰有 $p$ 个 1，后 $n-k$ 位中恰有 $\omega-p$ 个 1，则搜索 $\tilde{e}_1 \in \mathbb{F}_2^k$（满足 $\mathrm{wt}(\tilde{e}_1) = p$）即可恢复完整解。

随后，Lee-Brickell、Stern 等人不断优化。Stern (1989) 引入 **Meet-in-the-Middle (MITM)** 思想，将搜索空间分成两半并行构造列表，将复杂度降至 $2^{0.05564n}$。

**Ball-collision** (Bernstein, Lange & Peters, CRYPTO 2011) 允许非精确匹配，进一步降至 $2^{0.05559n}$。

**MMT** (May, Meurer & Thomae, Asiacrypt 2011) 引入**表示技术（representation technique）**：将解 $I$ 表示为两个不相交集合的并 $I = I_1 \cup I_2$，每个解有 $\binom{p}{p/2}$ 种表示，从而只需构造 $\binom{k+l}{p/2}$ 规模的列表（而非完整的 $\binom{k/2}{p/2}$），复杂度降至 $2^{0.05364n}$。

## 2. Submatrix Matching Problem (SMP)

MMT 将 ISD 的搜索阶段归结为**子矩阵匹配问题（Submatrix Matching Problem, SMP）**：

> 给定随机矩阵 $Q \in \mathbb{F}_2^{l \times (k+l)}$ 和目标向量 $s \in \mathbb{F}_2^l$，求集合 $I \subseteq [1, k+l]$，$|I|=p$，使得
> 
> $$\sigma(Q_I) := \sum_{i \in I} q_i = s,$$
> 
> 其中 $q_i$ 为 $Q$ 的第 $i$ 列。

SMP 可视为一个更小的伴随式解码实例（校验矩阵为 $Q$，伴随式为 $s$，参数为 $[k+l, l, p]$）。MMT 的核心思路是**表示**：将目标集合 $I$ 写成 $I = I_1 \cup I_2$，其中 $|I_1| = |I_2| = p/2$，$I_1 \cap I_2 = \varnothing$。这样，每个真实解 $I$ 对应 $\binom{p}{p/2}$ 种不同的 $(I_1, I_2)$ 配对，从而列表规模可以缩小 $\binom{p}{p/2}$ 倍。

## 3. 关键洞察：$1+1=0$

MMT 的局限在于：$I_1$ 和 $I_2$ 必须**不相交**，这意味着每个解位置（index）只能出现在其中一个集合中。换句话说，MMT 只能拆分 1-位：

$$
1 = 1+0 \quad \text{或} \quad 1 = 0+1.
$$

**BJMM 的核心创新**：打破不相交约束，允许 $I_1$ 和 $I_2$ **重叠**，设 $|I_1 \cap I_2| = \varepsilon > 0$。此时解集为**对称差**：

$$
I = I_1 \,\Delta\, I_2 = (I_1 \cup I_2) \setminus (I_1 \cap I_2).
$$

这带来了一个关键突破：**0-位也可以被拆分**！

- 0-位不在 $I$ 中，但可以同时出现在 $I_1$ 和 $I_2$ 中（即 $(1,1)$），由于 $1+1=0$ 在 $\mathbb{F}_2$ 中，其对对称差的贡献仍为 0。
- 同样地，0-位也可以不在 $I_1$ 和 $I_2$ 中（$(0,0)$），贡献也为 0。

因此，0-位的分配方式提供了额外的组合自由度。对于 $k+l-p$ 个不在 $I$ 中的位置，每个都有 $\binom{k+l-p}{\varepsilon}$ 种方式选入交集 $I_1 \cap I_2$。

> **表示数量的比较**：
> - MMT：每个解有 $\displaystyle \binom{p}{p/2}$ 种表示
> - BJMM：每个解有 $\displaystyle \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon}$ 种表示（增加因子 $\binom{k+l-p}{\varepsilon}$）

注意 $k+l-p \gg p$（因为码字中 0 的数量远大于 1 的数量），所以这个额外因子非常显著！

## 4. COLUMN MATCH: 三层算法

为了利用上述扩展表示，BJMM 提出了一个**三层（three-level）**的 divide-and-conquer 算法，结构类似于 Wagner 的广义生日悖论算法。

### 4.1 参数定义

$$
\begin{aligned}
p_1 &= \frac{p}{2} + \varepsilon_1, \\
p_2 &= \frac{p_1}{2} + \varepsilon_2 = \frac{p}{4} + \frac{\varepsilon_1}{2} + \varepsilon_2,
\end{aligned}
$$

其中 $\varepsilon_1, \varepsilon_2$ 是可调参数。表示数量为：

$$
R_1 = \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon_1}, \quad
R_2 = \binom{p_1}{p_1/2} \cdot \binom{k+l-p_1}{\varepsilon_2}.
$$

### 4.2 MERGE-JOIN 基础构建块

**MERGE-JOIN** 是整个算法的基础构建块，其功能是：给定两个列表 $L_1, L_2$（包含长度为 $k+l$ 的二进制向量），找出所有满足以下条件的配对 $(x, y) \in L_1 \times L_2$：

1. $\mathrm{wt}(x + y) = p$（重量约束）
2. $(Q(x+y))[r] = t$（前 $r$ 位匹配目标 $t$）

算法通过**排序 + 双指针**（借鉴 Knuth 的碰撞搜索）高效实现这一配对过程，时间复杂度为 $O(\max\{|L_1|, |L_2|, C\})$，其中 $C$ 为碰撞数量（$C \approx |L_1||L_2|/2^r$）。

### 4.3 三层结构（Layer 3 → Layer 0）

```
Layer 3 (底层):  4 对基列表 Bi,1, Bi,2  (不相交拆分)
Layer 2:         L(2)_i = MERGE-JOIN(Bi,1, Bi,2, r2, p2, t(2)_i)  (i = 1..4)
Layer 1:         L(1)_j = MERGE-JOIN(L(2)_{2j-1}, L(2)_{2j}, r1, p1, t(1)_j)  (j = 1,2)
Layer 0 (顶层):  L       = MERGE-JOIN(L(1)_1, L(1)_2, l, p, s)  →  解
```

**Layer 3 — 基列表构造**：以 $L^{(2)}_1$ 为例。将 $[k+l]$ 随机划分为等大的两部分 $P_1, P_2$（$|P_1| = |P_2| = (k+l)/2$），构造两个不相交列表：

$$
\begin{aligned}
B_1 &= \{ y \in \mathbb{F}_2^{k+l} \mid \mathrm{wt}(y) = p_2/2,\; y_i = 0 \; \forall i \in P_2 \}, \\
B_2 &= \{ z \in \mathbb{F}_2^{k+l} \mid \mathrm{wt}(z) = p_2/2,\; z_i = 0 \; \forall i \in P_1 \}.
\end{aligned}
$$

然后调用 $\text{MERGE-JOIN}(B_1, B_2, r_2, p_2, t^{(2)}_1)$ 得到 $L^{(2)}_1$。

**Layer 2 — 合并为 $p_2$ 重量向量**：$L^{(2)}_i$ 中的每个向量重量为 $p_2$，代表第一层表示的一半。

**Layer 1 — 合并为 $p_1$ 重量向量**：将 $L^{(2)}_1 \times L^{(2)}_2$ 和 $L^{(2)}_3 \times L^{(2)}_4$ 分别做 MERGE-JOIN，得到重量为 $p_1$ 的列表 $L^{(1)}_1$ 和 $L^{(1)}_2$。

**Layer 0 — 最终匹配**：对 $L^{(1)}_1, L^{(1)}_2$ 做 MERGE-JOIN，目标是匹配全部 $l$ 位、重量为 $p$，输出即为 SMP 的解。

### 4.4 正确性说明

由于对称差操作的性质：对于任意 $I_1, I_2$ 满足 $|I_1| = |I_2| = p_1 + \varepsilon_1$，$|I_1 \cap I_2| = \varepsilon_1$，有

$$
|I_1 \Delta I_2| = |I_1| + |I_2| - 2|I_1 \cap I_2| = 2(p_1 - \varepsilon_1) = p.
$$

类似地，Layer 2 中的合并也保证 $|e^{(2)}_{2i-1} + e^{(2)}_{2i}| = p_2$。所以三层 MERGE-JOIN 逐步将表示合并回正确重量，最终输出的解恰好满足 SMP 的约束。

## 5. 复杂度分析

### 5.1 各层规模

设 $S_3 = \binom{(k+l)/2}{p_2/2}$ 为基列表大小。假设部分和均匀分布，则各层列表大小为：

$$
S_2 = \binom{k+l}{p_2} \cdot 2^{-r_2}, \quad S_1 = \binom{k+l}{p_1} \cdot 2^{-r_1},
$$

其中 $r_i \approx \log_2 R_i$ 是第 $i$ 层约束的坐标数。三次 MERGE-JOIN 的时间复杂度为：

$$
T = \max\{T_3, T_2, T_1\}.
$$

### 5.2 总体复杂度

整个 ISD 算法的期望运行时间为：

$$
T_{\text{ISD}} = P^{-1} \cdot T(p, l; \varepsilon_1, \varepsilon_2),
$$

其中迭代概率

$$
P = \frac{\binom{k+l}{p}\binom{n-k-l}{\omega-p}}{\binom{n}{\omega}}.
$$

通过数值优化参数 $(p, l, \varepsilon_1, \varepsilon_2)$，得到各复杂度系数：

| 算法 | 半距离解码 | 存储系数 |
|------|-----------|---------|
| Lee-Brickell | $2^{0.05752n}$ | — |
| Stern | $2^{0.05564n}$ | $2^{0.0135n}$ |
| Ball-collision | $2^{0.05559n}$ | $2^{0.0148n}$ |
| MMT | $2^{0.05364n}$ | $2^{0.0216n}$ |
| **BJMM** | $2^{0.04934n}$ | $2^{0.0286n}$ |

对于典型 McEliece 参数（相对距离 $D=0.04$，码率 $R=0.7577$），BJMM 的复杂度为 $2^{0.0672n}$，相较 MMT 的 $2^{0.0760n}$ 有约 $2^{0.0088n}$ 的指数改进。

## 6. 小结

BJMM 的核心贡献在于**扩展了表示技术**：通过允许索引集合重叠，巧妙利用了 $\mathbb{F}_2$ 中 $1+1=0$ 的代数性质，将 0-位的组合自由度纳入表示空间。这一洞察看似简单（"我们之前只拆了 1，没想到 0 还能拆"），但实际效果显著——将解码复杂度从 $2^{0.05364n}$ 降至 $2^{0.04934n}$。

后续工作（如 sieving-based ISD）进一步将复杂度压低，但 BJMM 的三路分解框架和"1+1=0"的思想，仍是理解现代 ISD 算法演进的必经之路。

## References

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in $2^{n/20}$: How $1+1=0$ Improves Information Set Decoding*. Eurocrypt 2012. ([ePrint 2012/026](https://eprint.iacr.org/2012/026))
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in $\tilde{O}(2^{0.054n})$*. Asiacrypt 2011.
4. D. J. Bernstein, T. Lange, C. Peters. *Smaller Decoding Exponents: Ball-Collision Decoding*. Crypto 2011.
