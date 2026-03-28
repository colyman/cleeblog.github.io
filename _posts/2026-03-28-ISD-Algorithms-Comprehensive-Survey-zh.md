---
layout: article
title: "ISD 算法全景综述与前沿进展（1962–2026）"
date: 2026-03-28
key: ISD-Algorithms-Comprehensive-Survey-1962-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, McEliece, Post-Quantum-Cryptography, Survey]
hidden: true
---

## tl;dr

本文系统梳理 **信息集解码（ISD）算法** 的六十年发展脉络（1962–2026）。ISD 是破解 McEliece 等码基密码系统的核心攻击手段，也是评估后量子密码安全性的基准工具。从 Prange 的基础框架到 Stern 的生日碰撞、从 MMT/BJMM 的表示法革命到 May–Ozerov 的 Walsh–Hadamard 变换加速、从 CCJ 到 Esser–Santini 的正规解码突破，本文以复杂度分析为主线，串联各代算法的核心思想、技术创新与局限性，并附完整参考文献。

**关键词**：ISD、McEliece、综合征解码、表示法攻击、后量子密码学

---

## 1. 引言：什么是信息集解码？

**信息集解码（Information Set Decoding, ISD）** 是一类用于解线性码的算法的总称。其目标是：

> 给定一个 $[n, k]$ 线性码的生成矩阵（或校验矩阵）$G$ 和一个码字 $y = c + e$（其中 $c \in \mathcal{C}$ 是合法码字，$e$ 是未知错误向量），在 $\|\!e\!\| \leqslant t$ 的条件下，恢复码字 $c$ 或等价的错误向量 $e$。

在密码学语境下，这等价于求解 **综合征解码问题（Syndrome Decoding Problem, SDP）**：

$$H \in \mathbb{F}_2^{(n-k)\times n}, \quad s \in \mathbb{F}_2^{n-k}, \quad w > 0$$
$$\text{求 } e \in \mathbb{F}_2^n \text{ 使得 } \mathrm{wt}(e) \leqslant w \text{ 且 } eH^\top = s$$

SDP 由 Berlekamp–McEliece–van Tilborg 于 1978 年证明是 **NP完全** 的。这意味着对于一般线性码，不存在已知的多项式时间算法。

### 1.1 为什么 ISD 如此重要？

McEliece 于 1978 年提出的 **McEliece 公钥密码系统** 是最早的基于编码的密码系统。其安全性直接依赖于 ISD 的难度：攻击者需要从公开的生成矩阵 $G'$ 中恢复出错误向量 $e$，而这等价于解决 SDP。

六十年来，密码学社区投入了大量精力来改进 ISD 算法，以更准确地评估 McEliece 类系统的安全性。与此同时，这些改进也推动了密码系统参数的升级——这是一场密码学界的"军备竞赛"。

### 1.2 历史时间线

```
1962  Prange           ISDu（首个ISD算法）
1988  Lee-Brickell     允许 info set 中 p 个非零位
1989  Stern            生日碰撞（birthday collision）
1998  Canteaut-Chabaud 邻近信息集迭代
2009  Finiasz-Sendrier 统一分析框架
2011  MMT              多重表示法（Multiple Representations）
2012  BJMM             "1+1=0" 突破 → 2^{0.04934n}
2015  May-Ozerov       Walsh-Hadamard 最近邻加速
2023  CCJ              正规SD的 CCJ-1/2 算法
2024  Esser-Santini    Regular-ISD → 2^{0.112n}
2025  多项前沿工作      扩展域ISD、卷积码ISD、QC-LDPC比较研究
```

---

## 2. 标准 ISD：从 Prange 到 Stern

### 2.1 Prange ISDu（1962）

Prange 于 1962 年提出了 **信息集解码（ISDu）** 的基本框架。其核心思想异常简洁：

1. 随机选择一个大小为 $n-k$ 的**信息集** $I \subset \{1, \ldots, n\}$（即 $G$ 的线性无关列）
2. 对 $G$ 做高斯消元得到系统形式 $[I_k | A]$
3. 如果错误向量 $e$ 在信息集 $I$ 中的非零位置数量很少，则可以从 $y$ 的对应部分直接恢复 $e$

设 $p = \mathrm{wt}(e \cap I)$ 为 $e$ 在信息集中的非零位数。若 $p = 0$（即 $e$ 的所有非零位都在信息集之外），则 $e$ 可以被直接读出。

**工作量估计**：
$$\text{WF} \approx \binom{n}{w}^{-1} \cdot \binom{n-k}{w}$$

这是后续所有 ISD 算法的**起点和基准**。

### 2.2 Lee–Brickell 算法（1988）

Lee 和 Brickell 的关键改进在于：允许信息集中存在少量非零位。

设 $e$ 在信息集中恰好有 $p$ 个非零位（其余 $w-p$ 个非零位在冗余部分）。Lee–Brickell 算法通过**局部搜索**处理这 $p$ 个非零位：

- 随机选择 $p$ 个信息集位置，**翻转**这些位置（对应在 $e$ 中置入非零）
- 对冗余部分应用类似 Prange 的策略

**复杂度**：
$$\text{WF}_{\text{LB}}(n, k, w) \approx \min_{p} \binom{k}{p} \cdot \binom{n-k}{w-p}$$

对于典型 McEliece 参数 $[n, n/2]$，Lee–Brickell 给出 $\text{WF} \approx 2^{0.05752n}$。

**意义**：Lee–Brickell 表明，即使允许信息集中存在少量非零位，ISD 仍然高效——这使得"简单增加冗余位"并不能显著提升 McEliece 的安全性。

### 2.3 Stern 算法（1989）—— 生日碰撞

Stern 于 1989 年提出了最具影响力的改进，其核心思想是 **生日碰撞**。

**算法框架**：
1. 将错误向量 $e$ 分解为两部分：$e_1$（$k$ 维）和 $e_2$（$n-k$ 维）
2. 设 $e_1$ 有 $p$ 个非零位，$e_2$ 有 $p$ 个非零位（各占一半）
3. 构造列表 $L_1$ 和 $L_2$，分别包含所有可能的 $e_1$ 和 $e_2$
4. **生日碰撞**：寻找 $(e_1, e_2) \in L_1 \times L_2$ 使得 $e_1 \oplus e_2 = s$（给定伴随式）

具体实现中，两列表各有约 $\binom{k/2}{p/2} \approx 2^{k \cdot H(p/k/2)}$ 个元素。碰撞概率为 $2^{-(n-k)}$，因此期望列表大小为 $2^{(n-k)/2}$。

**Stern 复杂度公式**：
$$\text{WF}_{\text{Stern}} \approx \min_p \binom{k}{p} \cdot \binom{n-k}{w-p} \cdot 2^{-\frac{n-k}{2}} \cdot 2^{\frac{k}{2}} = 2^{0.05563n} \text{（渐近）}$$

**核心洞察**：Stern 将"暴力搜索"问题转化为"生日碰撞"问题，将复杂度从 $O(2^n)$ 降低到 $O(2^{n/2})$，这是一个平方根的改进。

### 2.4 Canteaut–Chabaud 算法（1998）—— 邻近信息集

Stern 算法的每次迭代都需要重新做高斯消元（$O(n^3)$），这在工程实现中代价很高。

**Canteaut–Chabaud 的关键改进**是 **邻近信息集（Close Information Sets）**：

> 如果两个信息集 $I$ 和 $I'$ 仅相差一个元素（即 $|I \setminus I'| = |I' \setminus I| = 1$），则称它们为邻近信息集。

通过随机游走，可以在邻近信息集之间高效跳转，每次只需 $O(n)$ 的行交换操作（而非 $O(n^3)$ 的完整高斯消元）。

算法以 **马尔可夫链** 建模：状态为当前信息集中目标非零位的数量，每次跳转改变状态 $\pm 1$ 或 $0$。唯一吸收态为"成功态"（恰好 $p$ 个非零位落入信息集的特定子集）。

**结果**：对于 $[1024, 524, 50]$ Goppa 码，Canteaut–Chabaud 给出 $\text{WF} \approx 2^{64.2}$（相比之前改进）。

---

## 3. Finiasz–Sendrier 统一分析框架（2009）

Finiasz 和 Sendrier 于 2009 年（Asiacrypt）提供了 **所有已知 ISD 变体的统一下界分析**。

### 3.1 核心工具：Parikh 映射

他们引入了 **Parikh 向量** 的概念。设 $e \in \mathbb{F}_2^n$，其 Parikh 向量定义为各坐标位置集合的散列值：

$$h_\ell(x) = \sum_{i=0}^{\ell-1} x_i \cdot 2^i \mod 2^\ell$$

### 3.2 统一公式

对于任意 ISD 算法，工作因子满足：

$$\text{WF}_{\text{ISD}}(n, r, w) \approx \min_p 2^\ell \cdot \min\left(\binom{n}{w}, 2^r\right) \cdot \binom{r-\ell}{w-p} \cdot \sqrt{\binom{k+\ell}{p}}$$

其中 $\ell \approx \log(K_{w-p} \cdot \sqrt{\binom{k}{p}})$，$K_w \approx 2^{w \cdot H(w/k)}$ 是重量 $w$ 的向量数量估计。

### 3.3 生日常数 $\xi$

Finiasz–Sendrier 证明，对于二进制表示的碰撞搜索，最优碰撞权重为 $z=1$，这产生常数：

$$\xi = 1 - e^{-1} \approx 0.63212$$

该常数量化了"生日碰撞"中有效配对的概率。

**意义**：这些下界是**紧的**——没有新技术的引入，现有任何 ISD 算法都无法突破这些界。这为 McEliece 参数选择提供了可靠的基准。

---

## 4. 表示法攻击：从 MMT 到 BJMM

2011–2012 年出现了 ISD 历史上最重要的突破之一：**表示法攻击（Representation Attacks）**。

### 4.1 MMT 算法（2011）

MMT（May–Meurer–Thomae）于 2011 年提出：不再只考虑"一个元素等于 0 或 1"，而是允许每个元素有**多种表示**。

以有限域 $\mathbb{F}_2$ 为例：

$$1 = 0 \oplus 1 \quad \text{或} \quad 1 = 1 \oplus 0$$

这看似平凡，但当与**分裂（splitting）**技术结合时，就产生了强大效果：

- 将码字/错误向量分裂为多个小权重块
- 每块可以有多种表示方式
- 列表构造时指数增长，但合法解仍是线性的

**复杂度**：MMT 给出 $\text{WF} = 2^{0.05364n}$，相比 Stern 的 $2^{0.05563n}$ 有显著提升。

### 4.2 BJMM 算法（2012）—— "1+1=0" 的革命

Becker–Joux–May–Meurer 于 Eurocrypt 2012 提出了里程碑式的 **"1+1=0" 洞见**。

**核心观察**：在 $\mathbb{F}_2$ 上，不仅 $1$ 有两种表示，$0$ 也有两种表示：

$$0 = 0 \oplus 0 \quad \text{且} \quad 0 = 1 \oplus 1$$

这看似平凡的恒等式，实际上是 **BJMM 的关键**——它允许构造更多的"伪解"（phantom solutions），从而在碰撞阶段获得更大的列表。

### 4.3 三层 MERGE-JOIN 结构

BJMM 采用 **三层树结构** 来解决子矩阵匹配问题：

```
第三层（底层）：B₁,₁ B₁,₂ | B₂,₁ B₂,₂ | B₃,₁ B₃,₂ | B₄,₁ B₄,₂  （无交集基础列表）
第二层：          L⁽²⁾₁    L⁽²⁾₂    | L⁽²⁾₃    L⁽²⁾₄            （合并连接）
第一层：          L⁽¹⁾₁              L⁽¹⁾₂                      （合并连接）
顶层：            L⁽⁰⁾ = 答案                                 （合并连接）
```

每层的 MERGE-JOIN 操作将两个子列表连接，寻找特定汉明重量的匹配对。

**参数设计**：
- $p_1 = p/2 + \varepsilon_1$
- $p_2 = p_1/2 + \varepsilon_2 = p/4 + \varepsilon_1/2 + \varepsilon_2$
- 表示数：$R_1 = \binom{p}{p/2} \cdot \binom{k+\ell-p}{\varepsilon_1}$，$R_2 = \binom{p_1}{p_1/2} \cdot \binom{k+\ell-p_1}{\varepsilon_2}$

**BJMM 复杂度**：$\text{WF} = 2^{0.04934n}$

这意味着，对于 $n = 4000$ 的 McEliece 实例，BJMM 比 Prange 快约 $2^{20} \approx 100$ 万倍。

**历史意义**：BJMM 将 ISD 的复杂度"指数"从约 $0.055n$ 降到了 $0.049n$，这是持续三十年的重大突破。

---

## 5. 最近邻搜索：May–Ozerov 算法

### 5.1 Stern 算法的瓶颈

Stern 算法的核心步骤是 **排序-匹配（Sort-and-Match）**：构造两个列表 $L_1$ 和 $L_2$，寻找汉明距离为 $p$ 的配对。

**问题**：列表大小为 $2^{k/2}$，排序-匹配需要 $O(2^{k/2})$ 或 $O(2^{k/2} \log 2^{k/2})$ 时间。当 $k$ 很大时，这成为计算瓶颈。

### 5.2 Walsh–Hadamard 变换（WHT）

对于布尔函数 $f: \mathbb{F}_2^m \to \mathbb{F}_2$，其 **Walsh 变换** 定义为：

$$W_f(u) = \sum_{x \in \mathbb{F}_2^m} (-1)^{f(x) + u \cdot x}$$

WHT 的关键性质：**Parseval 恒等式**（能量守恒）。

快速 WHT 可以在 $O(m \cdot 2^m)$ 时间内计算所有 $2^m$ 个 Walsh 系数。

### 5.3 HIVe 算法（High-Independence Vectors）

May–Ozerov 的核心洞见是：用 **WHT 加速最近邻搜索**。

**设置**：给定两个集合 $U, V \subset \mathbb{F}_2^m$，目标为找到所有距离为 $\delta m$ 的配对。

**HIVe 步骤**：
1. 将向量分裂：$x \in \mathbb{F}_2^m \to (x_1, x_2)$，$x_1, x_2 \in \mathbb{F}_2^{m/2}$
2. 对每个可能的 $r \in \{0, \ldots, \delta m\}$：
   - 找出 $U$ 中 $x_1$ 权重为 $r$ 的所有向量（$2^{m_1}$ 个）
   - 找出 $V$ 中 $x_2$ 权重为 $\delta m - r$ 的所有向量（$2^{m_2}$ 个）
3. 对这两组向量应用 WHT，在 WHT 域中寻找碰撞

关键在于：汉明距离与 WHT 的相关性可以通过**卷积**高效处理。

### 5.4 应用于 ISD：Stern-MO 和 BJMM-MO

May–Ozerov 将 HIVe 替换 Stern/BJMM 中的排序-匹配步骤：

| 算法 | 复杂度（渐近） | 说明 |
|------|--------------|------|
| Stern | $2^{0.05563n}$ | 排序-匹配 |
| Stern-MO | $2^{0.05498n}$ | WHT 最近邻 |
| BJMM | $2^{0.04934n}$ | 排序-匹配 |
| BJMM-MO | 约 $2^{0.053n}$（近似） | WHT 最近邻 |

### 5.5 "Galactic" 算法（2025）

2025 年，Bouillaguet–Delaplace–Hamdad 在 *IACR Communications in Cryptology* 发表了一篇影响深远的论文，标题即为：**"The May–Ozerov Algorithm for Syndrome Decoding is 'Galactic'"**

**核心发现**：
- May–Ozerov 算法要优于 Stern 算法，交叉点在 **$n > 1{,}874{,}400$**
- 交叉点处操作数为 $2^{63{,}489}$ — 远超可观测宇宙中的原子数量

**结论**：对于所有实际相关的码大小（$n \lesssim 2^{20} \approx 100$ 万），Stern-MO 的常数开销使得原始 Stern 算法仍然更快。

这是一个重要的提醒：**渐近复杂度不是唯一标准**，多项式因子和常数在实践中同样关键。

---

## 6. 正规综合征解码（Regular ISD）

### 6.1 什么是正规综合征解码（RSD）？

正规综合征解码（RSD）由 Augot–Finiasz–Sendrier（AFS 2005）引入，是标准 SD 的结构化变体：

- 将向量 $e \in \mathbb{F}_2^n$ 分划为 $t$ 个等长块：$e = (e_1 \| e_2 \| \cdots \| e_t)$，每块长度 $N$
- 每块恰好有 1 个非零位：$\mathrm{wt}(e_i) = 1$（$i = 1, \ldots, t$）
- 总权重 $w = t$

**数学结构**：环 $\mathbb{F}_2[X]/(X^n - 1)$ 上的多项式方程：
$$h(X) \cdot e(X) \equiv s(X) \pmod{X^n - 1}$$

### 6.2 环分解与圆锥陪集

关键在于 $X^n - 1$ 的分解。令 $C_n = \{\text{在 } \mathbb{Z}_n \text{ 上由 } 2 \text{ 生成的圆锥陪集}\}$：

$$X^n - 1 = \prod_{C \in C_n} m_C(X)$$

通过中国剩余定理（CRT）：
$$\mathbb{F}_2[X]/(X^n-1) \cong \bigoplus_{C \in C_n} \mathbb{F}_2[X]/(m_C(X))$$

每个陪集 $C$ 的多项式环分量阶为 $|C|$。这将原始问题分解为多个子问题。

### 6.3 CCJ 算法（Eurocrypt 2023）

Carozza–Couteau–Joux 于 2023 年提出了两种 RSD 攻击算法：

| 算法 | 复杂度 | 核心方法 |
|------|--------|---------|
| CCJ-1 | $2^{0.141n}$ | 线性化 |
| CCJ-2 | $2^{0.135n}$ | 改进的线性化 |

**Esser–Santini 贡献（CRYPTO 2024）**：对 CCJ 算法进行了**首个严格的渐近分析**，验证了上述复杂度估计。

### 6.4 Regular-ISD（Esser–Santini，CRYPTO 2024）

Esser 和 Santini 的主要贡献是：将**完整的先进 ISD 技术栈**适配到 RSD 场景。

**核心思想**：
1. 应用 CRT 环分解，将问题分解到各圆锥陪集分量
2. 在每个分量上**独立运行 ISD**（该 ISD 受额外的正规约束）
3. 合并各分量的解以恢复完整错误向量

**结果**：

| 算法 | 复杂度 | 备注 |
|------|--------|------|
| CCJ-1 | $2^{0.141n}$ | 线性化 |
| CCJ-2 | $2^{0.135n}$ | 改进线性化 |
| **Regular-ISD** | **$2^{0.112n}$** | CRT分解 + 分量ISD |

相比 CCJ，Regular-ISD 提升了约 $2^{0.029n}$，即对于 $n=2000$，快了约 $2^{58}$ 倍。

**安全影响**：对 CCJ 类签名方案，建议参数需重新评估，部分设置可能降低最多 **30 bits** 的安全级别。

---

## 7. 最新进展与前沿（2024–2026）

### 7.1 量子 ISD

量子计算对 ISD 的影响是双重的：

**量子Sieving（Ducas–Esser–Etinski–Kirshanova 2024）**：利用量子计算中的 **Grover 搜索** 可以在无序集中找到匹配项，给出 $O(2^{(n-k)/3})$ 量级的改进（Grover 的平方根加速适用于多维搜索）。

**量子 ISD（Babbitt–Han–Krishna–Potluri 2024）**：在量子计算模型下，ISD 的复杂度可以进一步降低。但关键在于：**McEliece 类方案仍被认为是量子安全的**，因为即使应用量子 ISD，其开销也远不足以在实用时间内破解推荐参数。

### 7.2 扩展域上的 ISD（ePrint 2025/1402）

2025 年的一篇重要论文探讨了：**是否可以通过将问题提升到更大的有限域 $\mathbb{F}_q$（$q > 2$）来加速 ISD？**

**直觉**：$\mathbb{F}_q$ 上汉明重量的定义与 $\mathbb{F}_2$ 不同（单字符非零即计重量），更大的域可能提供更多结构。

**发现**：扩展域确实改变了问题的某些性质，但代价是检索空间以 $q$ 的对数增长。结论是：**对标准 McEliece 实例，使用 $\mathbb{F}_2$ 仍然是最优选择**。

### 7.3 卷积码上的 ISD（Springer 2025）

2025 年发表在 *Designs, Codes and Cryptography* 上的工作将 **ISD 框架推广到了卷积码**。

这扩展了 ISD 的适用范围：McEliece 型密码系统的公开密钥可以是卷积码的形式，而不仅是块码。

### 7.4 环-线性码上的 ISD（ACM 2025）

ACM *Transactions on Privacy and Security* 2025 年的工作研究了 **环-线性码（Ring-Linear Codes）上的信息集解码**。

核心思想：将解码问题投影到**更小字母表的表示**，这可能启用更高效的解码算法。

### 7.5 QC-LDPC McEliece 的 ISD 比较研究（2025）

2025 年发表在 *Finite Fields and Their Applications* 的研究系统比较了 **QC-LDPC（准循环低密度奇偶校验）码 McEliece 系统** 上的各类 ISD 攻击：

- 不同 ISD 算法在不同参数配置下的实际效率
- 与理论复杂度预测的对比验证
- 对 QC-LDPC McEliece 参数选择的具体建议

---

## 8. 完整复杂度总表

下表汇总了从 1962 年到 2026 年所有主要 ISD 算法的复杂度：

| 算法 | 年份 | 渐近复杂度 | 核心思想 |
|------|------|-----------|---------|
| Prange ISDu | 1962 | $\sim 2^{0.5n}$ | 随机信息集 |
| Lee-Brickell | 1988 | $2^{0.05752n}$ | info set 中允许 $p$ 个非零 |
| Stern | 1989 | $2^{0.05563n}$ | 生日碰撞 |
| Canteaut-Chabaud | 1998 | $\approx 2^{0.055n}$ | 邻近信息集 |
| Finiasz-Sendrier | 2009 | 统一框架 | Parikh 映射，$\xi \approx 0.63$ |
| MMT | 2011 | $2^{0.05364n}$ | 多重表示 |
| BJMM | 2012 | $2^{0.04934n}$ | **"1+1=0"**，三层 MERGE-JOIN |
| Stern-MO | 2015 | $2^{0.05498n}$ | WHT 最近邻 |
| CCJ-1 | 2023 | $2^{0.141n}$ | 正规 SD 线性化 |
| CCJ-2 | 2023 | $2^{0.135n}$ | 改进线性化 |
| Regular-ISD | 2024 | $2^{0.112n}$ | CRT 环分解 |

> **注**：上表的渐近指数均针对典型 McEliece 参数（$R = k/n \approx 0.5$，$D = d/n \approx 0.04$）。不同参数设置下，具体数值会有所变化。

---

## 9. McEliece 参数安全性

基于 BJMM 的最新分析，当前推荐参数的**预估安全级别**（以 80-bit 安全为基准）：

| 参数集 | $[n, k]$ | 估计工作因子 | 推荐用途 |
|--------|----------|-------------|---------|
| $[1024, 524]$ | 低安全 | $2^{80}$ | 旧标准 |
| $[2048, 1753]$ | 中安全 | $2^{142}$ | 推荐（2024）|
| $[2960, 2288]$ | 高安全 | $2^{192}$ | 长期安全 |
| $[4608, 4096]$ | 极高安全 | $2^{256}$ | 最高安全需求 |

**注意**：随着 Regular-ISD 的发展，部分基于正规 SD 的签名方案（如 CCJ 类）需要**重新上调参数**以维持同等安全级别。

---

## 10. 结语

六十年过去了，ISD 算法的发展映射了整个密码学界的演进轨迹：

- **1962–1989**：从基础到成熟，Prange → Lee–Brickell → Stern
- **2009–2012**：表示法革命，Finiasz–Sendrier → MMT → BJMM  
- **2015**：WHT 加速，虽然被贴上"Galactic"标签但拓宽了理论边界
- **2023–2024**：结构化解码，CCJ → Regular-ISD
- **2025–2026**：多域拓展，扩展域、卷积码、量子计算

**开放问题**：
1. 是否存在渐近优于 $2^{0.049n}$ 的 ISD 算法？
2. "Galactic" 问题（May–Ozerov）的多项式开销能否被消除？
3. 量子计算对 McEliece 安全性的实际影响有多大？
4. RSD 的参数安全边界是否已被充分理解？

---

## 参考文献

1. Prange, E. (1962). *The use of information sets in decoding cyclic codes*. IRE Transactions on Information Theory.
2. Lee, P.J., Brickell, E.F. (1988). *An Efficient Probabilistic Algorithm for Finding Minimum-Weight Words in a Linear Code*. Eurocrypt 1988.
3. Stern, J. (1989). *A Method for Finding Codewords of Small Weight*. Coding Theory and Applications.
4. Canteaut, A., Chabaud, F. (1998). *A New Algorithm for Finding Minimum-Weight Words in a Linear Code: Application to McEliece's Cryptosystem and to Narrow-Sense BCH Codes of Length 511*. IEEE Trans. IT.
5. Finiasz, M., Sendrier, N. (2009). *Security Bounds for the Design of Code-based Cryptosystems*. Asiacrypt 2009.
6. May, A., Meurer, A., Thomae, E. (2011). *Decoding Random Linear Codes in $\tilde{O}(2^{0.054n})$*. Asiacrypt 2011.
7. Becker, A., Joux, A., May, A., Meurer, A. (2012). *Decoding Random Binary Linear Codes in $2^{n/20}$: How $1+1=0$ Improves Information Set Decoding*. Eurocrypt 2012.
8. May, A., Ozerov, I. (2015). *On Computing Nearest Neighbors with Applications to Decoding of Binary Linear Codes*. Eurocrypt 2015.
9. Hirose, S. (2016). *May-Ozerov Algorithm for Nearest-Neighbor Problem over $\mathbb{F}_q$ and Its Application to ISD*. PQCrypto 2016.
10. Bouillaguet, C., Delaplace, C., Hamdad, M. (2025). *The May-Ozerov Algorithm for Syndrome Decoding is "Galactic"*. IACR Communications in Cryptology.
11. Carozza, C., Couteau, G., Joux, A. (2023). *A New Approach to Syndrome Decoding for Codes with Sparse Matrices*. Eurocrypt 2023.
12. Esser, A., Santini, P. (2024). *Not Just Regular Decoding: Asymptotics and Improvements of Regular Syndrome Decoding Attacks*. CRYPTO 2024.
13. Augot, D., Finiasz, M., Sendrier, N. (2005). *A Family of Fast Syndrome-Based Cryptographic Hash Functions*. Mycrypt 2005.
14. Ducas, L., Esser, A., Etinski, S., Kirshanova, E. (2024). *Quantum Sieving for Code-based Cryptanalysis*. ACISP 2023 / ePrint 2024/1358.

---

*本文为 ISD 算法系列综述（共 7 篇）。前 6 篇分别覆盖：Prange–Stern 基础、Finiasz–Sendrier 框架、BJMM 算法、May–Ozerov 最近邻、正规 SD 与 CCJ、以及 CCJ 邻近信息集技术。*
