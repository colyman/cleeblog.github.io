---
layout: article
title: "Finiasz–Sendrier ISD：编码密码学的安全界"
date: 2026-03-23
key: Finiasz-Sendrier-ISD-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, McEliece, Post-Quantum, ISD]
hidden: true
---

**tl;dr** Finiasz 和 Sendrier 在 2009 年的工作给出了 Information Set Decoding（ISD）算法的**统一分析框架**，推导出涵盖所有已知优化（包括 Stern、Canteaut-Chabaud、Bernstein-Lange-Peters 等）的**下界公式**。这一工作成为后来选择安全参数的事实标准，其下界与实际攻击的差距已经非常小。

<!-- more -->

## 1. 背景：为什么需要 ISD 下界？

McEliece 公钥密码系统自 1978 年提出以来，其安全性始终建立在**一般线性码的译码困难问题**（Syndrome Decoding Problem, SDP）之上。SDP 是 NP 完全的——但"困难"是渐近的，具体选多少位码字、纠多少错误才能达到 2^{128} 的安全级别，需要精确分析。

2009 年之前，已有若干针对 McEliece 的实际攻击实现：

| 工作 | 年份 | 攻击复杂度（二进制位运算） |
|------|------|--------------------------|
| Canteaut-Chabaud | 1998 | 2^{64.2} |
| Bernstein-Lange-Peters | 2008 | 2^{60.5} |
| 真实攻击报告 | 2008 | ~2^{58} CPU cycles |

但这些工作都针对**具体参数**实现，缺乏**统一的下界公式**——也就是说，如果未来有人做了新的优化，谁也不知道还有多少安全余量。

Finiasz 和 Sendrier 的目标：**给出所有已知 ISD 变体的下界**，使密码系统设计者能够"一次分析，长期有效"。

## 2. 译码问题形式化

### 2.1 计算译码问题（CSD）

**问题 1（Computational Syndrome Decoding）**  
给定矩阵 $H \in \mathbb{F}_2^{r \times n}$、向量 $s \in \mathbb{F}_2^r$ 和整数 $w > 0$，寻找向量 $e \in \mathbb{F}_2^n$，使得 $\mathrm{wt}(e) \leq w$ 且 $e H^\top = s$。

这是 McEliece 和 Niederreiter 加密系统的核心安全假设。

### 2.2 解的唯一性

在密码学中，CSD 实例通常有**恰好一个解**（如公钥加密场景），但当错误数 $w$ 超过 Gilbert-Varshamov 距离时，可能有**多个解**（如签名或哈希场景）。两种情况的复杂度分析有所不同。

## 3. 生日攻击：最基础的方法

生日攻击是 ISD 最简单的形式：将码字分成左右两半，利用生日悖论寻找碰撞。

设将 $H = (H_1 \mid H_2)$ 分块，构造集合：
$$L_1 = \{e_1 H_1^\top : e_1 \in W_{n/2, w/2}\}, \quad L_2 = \{s + e_2 H_2^\top : e_2 \in W_{n/2, w/2}\}$$

若 $e_1 + e_2$ 是解，则 $e_1 H_1^\top = s + e_2 H_2^\top$，即 $L_1 \cap L_2 \neq \emptyset$。碰撞概率为：
$$\Pr_{n,w} = \frac{\binom{n/2}{w/2}^2}{\binom{n}{w}} \approx \frac{4}{\sqrt{\pi w}}$$

在 $L = \min\left(\sqrt{\binom{n}{w}}, 2^{r/2}\right)$ 下，生日译码的工作因子下界为：
$$\mathrm{WF}_{BA}(n, r, w) \geq \sqrt{2} \cdot L \cdot \log_2(2L)$$

这已经比朴素的暴力枚举好得多，但生日攻击并非最优——Stern 的 ISD 在密码学参数范围内更有效。

## 4. Stern 的 ISD 算法

### 4.1 核心思想

Stern（1989）将**生日攻击**的思想引入传统 ISD：不是在全空间中搜索，而是先做**部分高斯消元**，然后在缩小后的空间中做 **ℓ 位碰撞搜索**。

关键创新：
1. 对 $(n-k) \times n$ 的校验矩阵 $H$ 进行随机置换 $P$
2. 执行**部分**高斯消元，得到特殊形式：
$$UH_0P = \begin{bmatrix} I_{r-\ell} & H_1 \\ 0 & H_2 \end{bmatrix}$$
3. 将解 $e$ 分块：在前 $r-\ell$ 位放 $p$ 个 1（可通过查表），在后 $\ell$ 位放剩余 $w-p$ 个 1（通过碰撞搜索找到）
4. 碰撞搜索：要求 $e_1 H_1^\top$ 和 $s + e_2 H_2^\top$ 在某 ℓ 位上相等

### 4.2 经典复杂度

Stern 原始算法的工作因子近似为：
$$T \approx \min_p \frac{1}{p!} \cdot \binom{n}{w}^{-1} \cdot \binom{r}{\ell, w-p, r-\ell-w+p} \cdot \binom{k}{\ell}^{-1}$$

这个公式是许多实际实现的依据，但缺乏**统一的下界**——不同的实现有不同的优化trade-offs。

## 5. Finiasz–Sendrier 的统一分析

### 5.1 广义 ISD 算法

论文在 **Table 2** 给出了广义 ISD 算法，其关键改进是：

1. **在校验矩阵上工作**：而非生成矩阵，这与 Leon 的概率算法和 Stern 的方法都兼容
2. **引入参数 `p` 和 `ℓ`**：$p$ 控制解中确定部分的大小，ℓ 控制碰撞搜索的位数
3. **使用部分高斯消元**（partial Gaussian elimination），消元本身在理想化模型中视为"免费"

算法流程（Table 2）：

```
ISDecoding(H_0, s_0):
    repeat (main loop)
        P ← random n×n permutation
        (H', U) ← P GElim(H_0 P)   // partial elimination
        s ← s_0 U^T
        for all e₁ ∈ W₁:
            i ← h_ℓ(e₁ H'^T)       // isd 1
            store(e₁, i)
        for all e₂ ∈ W₂:
            i ← h_ℓ(s + e₂ H'^T)   // isd 2
            S ← read(i)
            for all e₁ ∈ S:
                if wt(s + (e₁+e₂)H'^T) = w-p:  // isd 3
                    return (P, e₁+e₂)           // success
```

其中 $h_ℓ(x)$ 表示向量 $x$ 的前 ℓ 位。$W_1 \subseteq W_{k+\ell, \lceil p/2 \rceil}$，$W_2 \subseteq W_{k+\ell, \lfloor p/2 \rfloor}$。

### 5.2 关键假设

**假设 I1**：所有被检查的配对 $(e_1, e_2)$，其和 $e_1 + e_2$ 在 $W_{k+\ell, p}$ 中均匀独立分布。

**假设 I2**：算法执行成本近似为：
$$C \approx \ell \cdot N_1 + \ell \cdot N_2 + K_{w-p} \cdot N_3$$
其中 $K_{w-p}$ 是检查 $\mathrm{wt}(s + (e_1+e_2)H'^T) = w-p$ 的平均成本。

### 5.3 工作因子下界（Proposition 2）

在假设 I1 和 I2 下，单解情形（$\binom{n}{w} < 2^r$）的工作因子为：

$$\boxed{\mathrm{WF}_{ISD}(n, r, w) \approx \min_p 2^\ell \cdot \min\!\left(\binom{n}{w}, 2^r\right) \cdot \xi \cdot \sqrt{\binom{r-\ell}{w-p}} \cdot \sqrt{\binom{k+\ell}{p}}}$$

其中：
- $\ell \approx \log_2\left(K_{w-p} \cdot \sqrt{\binom{k}{p/2}}\right)$
- $\xi = 1 - e^{-1} \approx 0.63$（来自 $z=1$ 时 $\xi(z) = 1-e^{-z}$ 的最优值）
- $p$ 和 ℓ 是需要优化的参数

**多解情形**（$\binom{n}{w} > 2^r$）有类似公式，但以 $2^{r/2}$ 为主项。

### 5.4 与 Stern 原始公式的比较

Stern 原始算法的工作因子：
$$T_{Stern} \approx \min_p 2^\ell \cdot \binom{n}{w} \cdot \binom{r-\ell}{w-p} \cdot \binom{k/2}{p/2}$$

新版本引入 $\xi \approx 0.63$ 和 $\sqrt{\binom{k+\ell}{p}}$ 代替 $\binom{k/2}{p/2}$，增益约为 $\xi \cdot 4\sqrt{\pi p/2}$，在实践中这是一个较小的常数改进。

**真正重要的不是常数改进，而是这个下界涵盖了所有已知优化变体**——无论实现者如何调参，工作因子都不会低于这个值。

## 6. 实际攻击结果

论文给出了针对 McEliece 标准参数的下界（Table 3）：

| 参数 $(m, w)$ | 最优 $p$ | 最优 ℓ | 二进制工作因子 |
|-------------|---------|--------|--------------|
| (10, 50) | 4 | 22 | $2^{59.9}$ |
| (11, 32) | 6 | 33 | $2^{86.8}$ |
| (12, 41) | 10 | 54 | $2^{128.5}$ |

表中参数对应码长 $n = 2^m$，维数 $k = n - mw$，校验矩阵 $r = mw$。

对于经典的 McEliece 参数 $(1024, 524)$，纠 $w = 50$ 个错误：
- Finiasz-Sendrier 下界：$2^{59.9}$
- Bernstein-Lange-Peters 实际实现：$2^{60.5}$
- 实际攻击（BLP 2008）：约 $2^{58}$ CPU cycles

**下界与实际最优攻击之间的差距已经非常小**，这意味着进一步优化空间极为有限——如果要攻破 McEliece，需要全新的技术。

## 7. 为什么这个下界如此重要？

### 7.1 面向设计者的工具

传统的密码分析是"从攻击者视角"给出上界，而 Finiasz-Sendrier 的工作提供了**从设计者视角**的下界：
- 下界是**持久的**：即使未来有人发现新的优化技巧，只要不引入全新技术，下界依然有效
- 设计者可以**一次分析，长期受益**：不必每次有新的攻击实现就重新评估参数

### 7.2 涵盖 GBA

论文同样分析了 Wagner 的 **广义生日算法**（GBA），适用于**多解情形**（如 FSB 哈希函数和 CFS 签名）。GBA 的下界为：
$$\mathrm{WF}_{GBA}(n, r, w) \geq \frac{r-a}{a} \cdot 2^{\frac{r-a}{a}}$$
其中 $a$ 满足 $\frac{1}{2^a}\binom{n}{w/2^a} = 2^{\frac{r-a}{a}}$。

### 7.3 启示

1. **McEliece 加密**：单解 → 用 ISD，$2^{60}$ 量级（经典参数）→ 需要更大参数
2. **CFS 签名**：多解 → GBA 更强 → 必须选用特殊设计的 Goppa 码
3. **FSB 哈希**：多解 → GBA 威胁，但实际攻击受限于输入约束（regular words）

## 8. 结论

Finiasz-Sendrier 2009 的工作建立了编码密码学参数选择的基础框架。通过给出 ISD 和 GBA 的**统一下界**，他们使设计者能够：

1. **准确评估安全裕度**：下界与实际最优攻击的差距即安全裕度
2. **持久化安全分析**：下界不随具体实现变化而失效
3. **跨系统比较**：McEliece、Niederreiter、CFS、FSB 都可以用同一套工具分析

**tl;dr 续**：原文结论中有一句话值得铭记："Solving CSD more efficiently than these bounds would require introducing new techniques, never applied to code-based cryptosystems." 这句话奠定了后来所有基于码的 PQC 标准的信心基础。

## 参考文献

1. M. Finiasz, N. Sendrier. *Security Bounds for the Design of Code-based Cryptosystems*. Asiacrypt 2009. https://eprint.iacr.org/2009/414
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. Canteaut, F. Chabaud. *A new algorithm for finding minimum-weight words in a linear code*. IEEE Trans. IT, 1998.
4. D. Bernstein, T. Lange, C. Peters. *Attacking and defending the McEliece cryptosystem*. PQCrypto 2008.
5. D. Wagner. *A generalized birthday problem*. Crypto 2002.
