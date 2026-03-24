---
layout: article
title: "BJMM Algorithm"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
bilingual: true
mathjax: true
---

**概要:** BJMM 算法（Becker-Joux-May-Meurer，Eurocrypt 2012）将随机线性码的 ISD 复杂度从 \\(2^{0.05364n}\\)（MMT）降至 \\(2^{0.04934n}\\)，是有史以来首个将复杂度指数压至 \\(0.05n\\) 以下的 ISD 算法。其核心思想是**允许索引集合重叠**（\\(|I_1 \cap I_2| = \varepsilon\\)），从而利用 \\(\mathbf{F}_2\\) 域中 \\(1+1=0\\) 的性质同时拆分 0-位和 1-位，显著增加每个解的表示数量。

Useful references:

- [Becker, Joux, May, Meurer — Eurocrypt 2012](https://eprint.iacr.org/2012/026)
- [May, Meurer, Thomae — Asiacrypt 2011 (MMT)](https://eprint.iacr.org/2011/605)
- [Stern — Coding Theory 1989](https://doi.org/10.1007/3-540-51280-8_14)

---

## 1. Background: Information Set Decoding (ISD)

设 \\(C\\) 为随机二元线性 \\([n, k, d]\\) 码，校验矩阵 \\(H \in \mathbf{F}_2^{(n-k)\times n}\\)。给定综合征 \\(s = He\\)，错误向量 \\(e \in \mathbf{F}_2^n\\) 权重 \\(\omega = \lfloor (d-1)/2 \rfloor\\)（半距离解码），**综合征解码问题**要求从 \\(s\\) 恢复 \\(e\\)。该问题是 McEliece 公钥密码系统安全性的核心假设。

### 1.1 Prange ISD 框架（1962）

1. **初始变换**：随机置换 \\(H\\) 的列（左乘可逆 \\(T\\)，右乘置换 \\(P\\)），化为准系统形式

$$\widetilde{H} = T \cdot H \cdot P = \begin{pmatrix} Q & I_{n-k} \end{pmatrix},\quad Q \in \mathbf{F}_2^{(n-k)\times k}.$$

设 \\(\tilde{e} = Pe\\)，成功概率为

$$P = \frac{\binom{k}{p}\binom{n-k}{\omega-p}}{\binom{n}{\omega}}.$$

2. **搜索阶段**：找到截断错误 \\(\tilde{e}_1 \in \mathbf{F}_2^k\\)（\\(p\\) 个 1），使 \\(Q\tilde{e}_1 = \tilde{s}\\). 这即**子矩阵匹配问题（SMP）**。

### 1.2 ISD 算法复杂度演进

| 算法 | 复杂度指数 | 核心技术 |
|------|-----------|---------|
| Lee-Brickell (1988) | \\(2^{0.05752n}\\) | \\(p < \omega\\) 优化 |
| Stern (1989) | \\(2^{0.05564n}\\) | 中途相遇（MITM） |
| Ball-collision (2011) | \\(2^{0.05559n}\\) | 非精确匹配 |
| MMT (2011) | \\(2^{0.05364n}\\) | 表示技术，不相交集合 |
| **BJMM (2012)** | **\\(2^{0.04934n}\\)** | 重叠集合，\\(1+1=0\\) |

## 2. 子矩阵匹配问题（SMP）

给定 \\(Q \in \mathbf{F}_2^{\ell \times (k+\ell)}\\)（\\(\ell = n-k\\)）和目标向量 \\(s \in \mathbf{F}_2^\ell\\)，求 \\(I \subseteq [k+\ell]\\)，\\(|I| = p\\)，使

$$\sigma(Q_I) := \sum_{i \in I} q_i = s.$$

SMP 本身就是一组参数为 \\([k+\ell, \ell, p]\\) 的综合征解码实例。改善 SMP 即改善 ISD。

## 3. MMT 的表示技术及其局限

MMT（Asiacrypt 2011）将解集 \\(I\\) 分解为两个**不相交**集合 \\(I = I_1 \cup I_2\\)，\\(|I_1| = |I_2| = p/2\\)，\\(I_1 \cap I_2 = \emptyset\\)。

- 每个解有 \\(\binom{p}{p/2}\\) 种表示（MMT）
- 局限：**仅拆分 1-位**（每个 1 为 \\((1,0)\\) 或 \\((0,1)\\))

## 4. 核心洞察：\\(1+1=0\\) 在 \\(\mathbf{F}_2\\) 中的应用

BJMM 认识到，利用 \\(1+1=0\\) 可以**同时拆分 0-位**：

| 原始位 | 拆分方式 | 求和贡献 |
|--------|---------|---------|
| 1 | \\((0, 1)\\) 或 \\((1, 0)\\) | ✓ MMT 已用 |
| **0** | \\((0, 0)\\) 或 \\((1, 1)\\) | **✓ BJMM 新增** |

由于 \\(\omega \ll n\\)，错误向量中 0 的数量远多于 1。允许 0-位拆分后，每个解的表示数量大幅增加。

### 重叠索引集合

BJMM 选择 \\(|I_1| = |I_2| = p/2 + \varepsilon\\)，\\(|I_1 \cap I_2| = \varepsilon\\)。通过**对称差**恢复：

$$I = I_1 \;\Delta\; I_2 = (I_1 \cup I_2) \setminus (I_1 \cap I_2).$$

交集 \\(I_1 \cap I_2\\) 中的元素同时出现在两侧，贡献 \\(1+1=0\\)，在列求和时相互抵消。

第一层表示数量：

$$R_1(p, \ell; \varepsilon_1) = \binom{p}{p/2} \cdot \binom{k+\ell-p}{\varepsilon_1}.$$

相比 MMT 增加了因子 \\(\binom{k+\ell-p}{\varepsilon_1}\\)。

两层表示总数：

$$R_2(p, \ell; \varepsilon_1, \varepsilon_2) = \binom{p}{p/2} \cdot \binom{k+\ell-p}{\varepsilon_1} \cdot \binom{p/2+\varepsilon_1}{p/4+\varepsilon_2/2} \cdot \binom{k+\ell-p/2-\varepsilon_1}{\varepsilon_2}.$$

## 5. MERGE-JOIN：核心操作

**MERGE-JOIN**\\(L_1, L_2, r, p, t\\) 返回：

$$L = \{ x + y \mid x \in L_1, y \in L_2,\ \mathrm{wt}(x+y) = p,\ (Q(x+y))[r] = t \}.$$

**匹配机制**：将 \\(L_1\\) 按标签 \\((Qx)[r]\\) 排序，\\(L_2\\) 按 \\((Qy)[r] + t\\) 排序，用双指针线性扫描找碰撞对。过滤不一致解（权重 \\(\neq p\\)）和重复元素。

运行时间：\\(\widetilde{O}(\max\{|L_1|, |L_2|, C\})\\)，\\(\mathbb{E}[C] = |L_1| \cdot |L_2| \cdot 2^{-r}\\).

### MERGE-JOIN Python 实现

```python
def merge_join(L1, L2, Q, r, p, t):
    """
    MERGE-JOIN: find pairs (x, y) with wt(x+y)=p and (Q(x+y))[r]=t.

    :param L1: list of vectors (list of int bits)
    :param L2: list of vectors
    :param Q: binary matrix as list-of-lists
    :param r: number of constrained coordinates
    :param p: target Hamming weight
    :param t: target vector (list of int bits)
    :return: list of result vectors
    """
    # Sort L1 by projected label Qx[r]
    L1_sorted = sorted(L1, key=lambda x: proj(x, Q, r))
    # Sort L2 by projected label Qy[r] + t
    L2_sorted = sorted(L2, key=lambda y: xor_proj(y, Q, r, t))

    results = []
    i = j = 0
    while i < len(L1_sorted) and j < len(L2_sorted):
        x, y = L1_sorted[i], L2_sorted[j]
        lx = proj(x, Q, r)
        ly = proj(y, Q, r)

        if lx < ly:
            i += 1
        elif lx > ly:
            j += 1
        else:
            xy = [a ^ b for a, b in zip(x, y)]
            if weight(xy) == p:
                results.append(xy)
            i += 1
            j += 1

    return results


def proj(v, Q, r):
    """Compute (Qv)[0:r] as integer."""
    return sum((dot(v, Q[row][:len(v)]) & 1) << r - 1 - row
               for row in range(r))


def xor_proj(v, Q, r, t):
    return proj(v, Q, r) ^ bits_to_int(t[:r])


def weight(v):
    return sum(v)


def dot(v, row):
    return sum(a & b for a, b in zip(v, row)) & 1


def bits_to_int(bits):
    return sum(b << i for i, b in enumerate(bits))
```

## 6. COLUMN MATCH：三层计算树

BJMM 通过三层 MERGE-JOIN 树求解 SMP，结构类似 Wagner 广义生日问题。

### 参数设置

| 参数 | 定义 |
|------|------|
| \\(p_1\\) | \\(p/2 + \varepsilon_1\\) |
| \\(p_2\\) | \\(p_1/2 + \varepsilon_2 = p/4 + \varepsilon_1/2 + \varepsilon_2\\) |
| \\(r_1\\) | \\(\approx \log_2 R_1\\) |
| \\(r_2\\) | \\(\approx \log_2 R_2\\) |

### 算法步骤

**第 3 层**：对 \\(i = 1, \ldots, 4\\)，随机划分 \\([k+\ell] = P_{i,1} \cup P_{i,2}\\)，构造不相交基列表

$$B_{i,1} = \{ y \mid \mathrm{wt}(y) = p_2/2,\ y_j = 0\ (j \in P_{i,2}) \}$$

$$B_{i,2} = \{ z \mid \mathrm{wt}(z) = p_2/2,\ z_j = 0\ (j \in P_{i,1}) \}$$

然后 \\(L^{(2)}_i = \text{MERGE-JOIN}(B_{i,1}, B_{i,2}, r_2, p_2, t^{(2)}_i)\\).

**第 2 层**：合并

$$L^{(1)}_1 = \text{MERGE-JOIN}(L^{(2)}_1, L^{(2)}_2, r_1, p_1, t^{(1)}_1)$$

$$L^{(1)}_2 = \text{MERGE-JOIN}(L^{(2)}_3, L^{(2)}_4, r_1, p_1, t^{(1)}_2)$$

**第 1 层（最终）**：合并得到解

$$L = \text{MERGE-JOIN}(L^{(1)}_1, L^{(1)}_2, \ell, p, s)$$

计算树结构：

```
第3层:  B1,1  B1,2 | B2,1  B2,2 | B3,1  B3,2 | B4,1  B4,2
         ───────────────   ───────────────   ───────────────   ─────────
第2层:      L(2)_1            L(2)_2            L(2)_3            L(2)_4
               ───────────────   ───────────────
第1层:           L(1)_1                  L(1)_2
               ────────────────────────────────
第0层:                      L  →  SMP 解
```

### COLUMN MATCH Python 实现

```python
def column_match(Q, s, p, eps1, eps2):
    """
    COLUMN MATCH: solve SMP (Submatrix Matching Problem) via BJMM.

    :param Q: binary matrix (list-of-lists, Q[row][col])
    :param s: target syndrome vector
    :param p: target weight
    :param eps1: overlap size at layer 1
    :param eps2: overlap size at layer 2
    :return: list of solution vectors e with wt(e)=p and Qe=s
    """
    k = len(Q[0])
    ell = len(Q)
    k_ell = k + ell

    p1 = p // 2 + eps1
    p2 = p1 // 2 + eps2

    # Layer 3: build 4 disjoint base list pairs
    L2_lists = []
    for i in range(4):
        P1, P2 = random_partition(k_ell)
        B1 = build_base_list(P1, P2, p2 // 2, k_ell)
        B2 = build_base_list(P2, P1, p2 // 2, k_ell)
        t2 = random_bits(r2_val)
        L2 = merge_join(B1, B2, Q, r2_val, p2, t2)
        L2_lists.append(L2)

    # Layer 2: merge pairs to form L(1)
    t1 = random_bits(r1_val)
    L1_1 = merge_join(L2_lists[0], L2_lists[1], Q, r1_val, p1, t1)
    L1_2 = merge_join(L2_lists[2], L2_lists[3], Q, r1_val, p1,
                      xor_bits(t1, s[:r1_val]))

    # Layer 1: final merge
    solutions = merge_join(L1_1, L1_2, Q, ell, p, s)
    return solutions


def random_partition(n):
    """Randomly split [0..n-1] into two equal halves."""
    idx = list(range(n))
    random.shuffle(idx)
    return idx[:n//2], idx[n//2:]


def build_base_list(pos_idx, zero_idx, weight_half, k_ell):
    """Build base list B where positions in pos_idx have weight bits."""
    vectors = []
    for combo in combinations(pos_idx, weight_half):
        v = [0] * k_ell
        for pos in combo:
            v[pos] = 1
        vectors.append(v)
    return vectors
```

## 7. 复杂度分析

三层 MERGE-JOIN 的时间复杂度：

$$T(p, \ell; \varepsilon_1, \varepsilon_2) = \max\{ T_3, T_2, T_1 \} = \max\{ S_3, C_3, S_2, C_2, S_1, C_1 \}$$

其中 \\(S_i = \binom{k+\ell}{p_i} \cdot 2^{-r_i}\\)，\\(\mathbb{E}[C_i] = S_i^2 \cdot 2^{r_i - r_{i-1}}\\)。

迭代成功概率：\\(P^{-1} = \frac{\binom{n}{\omega}}{\binom{k+\ell}{p}\binom{n-k-\ell}{\omega-p}}\\)。

在随机线性码上优化参数后：

$$T_{\text{BJMM}} = 2^{0.04934n} \quad \text{（半距离解码，最坏情况）}$$

存储复杂度：\\(\approx 2^{0.0286n}\\).

**McEliece 典型参数**（\\(R = 0.7577\\), \\(D = 0.04\\)）：

| 指标 | 半距离解码 | 完全解码 |
|------|-----------|----------|
| 时间 | \\(2^{0.0672n}\\) | \\(2^{0.1019n}\\) |
| 空间 | \\(2^{0.0586n}\\) | \\(2^{0.0769n}\\) |

## 8. 与前人工作对比

| 算法 | \\(2^{\alpha n}\\)（半距离） | 关键技术 |
|------|--------------------------|---------|
| Lee-Brickell | \\(2^{0.05752n}\\) | \\(p < \omega\\) |
| Stern | \\(2^{0.05564n}\\) | MITM |
| Ball-collision | \\(2^{0.05559n}\\) | 非精确匹配 |
| MMT | \\(2^{0.05364n}\\) | 表示，不相交 |
| **BJMM** | **\\(2^{0.04934n}\\)** | 重叠，\\(1+1=0\\) |

BJMM 相比 MMT 提升 \\(2^{0.0043n}\\) 倍，\\(n=2000\\) 时约 \\(3\times\\) 加速。

## 9. PROVABLE CM：可证明变体

启发式分析依赖 \\((Qe)[r]\\) 的均匀性。**PROVABLE CM** 使用新鲜随机目标值反复调用 COLUMN MATCH，当任意列表超过期望大小 \\(2^{\gamma n}\\) 倍时中止（常数 \\(\gamma > 0\\)). 除可忽略一部分矩阵 \\(Q\\) 外，该变体以 \\(> 1 - 3/e^2\\) 概率成功。

## 参考文献

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in \\(2^{n/20}\\): How \\(1+1=0\\) Improves Information Set Decoding.* Eurocrypt 2012. ([ePrint](https://eprint.iacr.org/2012/026))
2. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in \\(\tilde{O}(2^{0.054n}\\)).* Asiacrypt 2011.
3. J. Stern. *A method for finding codewords of small weight.* Coding Theory 1989.
