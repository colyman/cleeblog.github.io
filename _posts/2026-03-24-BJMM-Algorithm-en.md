---
layout: article
title: "BJMM Algorithm: How 1+1=0 Improves Information Set Decoding"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: en
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
bilingual: true
mathjax: true
---

**tl;dr** BJMM (Eurocrypt 2012) reduces ISD complexity to \\(2^{0.04934n}\) — the first algorithm to break the \\(0.05n\) barrier. The key insight: allowing index sets to overlap and exploiting \\(1+1=0\) in \\(\mathbf{F}_2\) to simultaneously split zero and one positions. This yields a three-layer merge-join tree (COLUMN MATCH) that solves the Submatrix Matching Problem more efficiently than MMT.

<!-- more -->

## 1. Background: Information Set Decoding (ISD)

Let \\(C\) be a random binary linear \\([n, k, d]\) code with parity-check matrix \\(H \in \mathbf{F}_2^{(n-k)\times n}\). Given syndrome \\(s = He\), the **syndrome decoding problem** asks to recover error vector \\(e \in \mathbf{F}_2^n\) of weight \\(\omega = \lfloor (d-1)/2 \rfloor\) (half-distance decoding). This problem underlies the security of the McEliece public-key cryptosystem.

### 1.1 Prange ISD Framework (1962)

1. **Initial permutation**: Randomly permute columns of \\(H\) (left-multiply invertible \\(T\), right-multiply permutation \\(P\)), transforming to quasi-systematic form

$$\widetilde{H} = T \cdot H \cdot P = \begin{pmatrix} Q & I_{n-k} \end{pmatrix},\quad Q \in \mathbf{F}_2^{(n-k)\times k}.$$

Let \\(\tilde{e} = Pe\), with success probability

$$P = \frac{\binom{k}{p}\binom{n-k}{\omega-p}}{\binom{n}{\omega}}.$$

2. **Search phase**: Find truncated error \\(\tilde{e}_1 \in \mathbf{F}_2^k\) (with \\(p\) ones) such that \\(Q\tilde{e}_1 = \tilde{s}\). This is the **Submatrix Matching Problem (SMP)**.

### 1.2 ISD Algorithm Complexity Evolution

| Algorithm | Complexity Exponent | Core Technique |
|-----------|-------------------|----------------|
| Lee-Brickell (1988) | \\(2^{0.05752n}\) | \\(p < \omega\) optimization |
| Stern (1989) | \\(2^{0.05564n}\) | Meet-in-the-middle (MITM) |
| Ball-collision (2011) | \\(2^{0.05559n}\) | Inexact matching |
| MMT (2011) | \\(2^{0.05364n}\) | Representation technique, disjoint sets |
| **BJMM (2012)** | **\\(2^{0.04934n}\)** | Overlapping sets, \\(1+1=0\) |

## 2. Submatrix Matching Problem (SMP)

Given \\(Q \in \mathbf{F}_2^{\ell \times (k+\ell)}\) (\\(\ell = n-k\)) and target vector \\(s \in \mathbf{F}_2^\ell\), find \\(I \subseteq [k+\ell]\), \\(|I| = p\), such that

$$\sigma(Q_I) := \sum_{i \in I} q_i = s.$$

SMP is itself a syndrome decoding instance with parameters \\([k+\ell, \ell, p]\). Improving SMP directly improves ISD.

## 3. MMT Representation Technique and Its Limitations

MMT (Asiacrypt 2011) decomposes solution set \\(I\) into two **disjoint** sets \\(I = I_1 \cup I_2\), \\(|I_1| = |I_2| = p/2\), \\(I_1 \cap I_2 = \emptyset\).

- Each solution has \\(\binom{p}{p/2}\) representations (MMT)
- Limitation: **only splits 1-bits** (each 1 becomes \\((1,0)\) or \\((0,1)\))

## 4. Core Insight: \\(1+1=0\) in \\(\mathbf{F}_2\)

BJMM leverages \\(1+1=0\) to **simultaneously split 0-bits**:

| Original Bit | Splitting Choices | Sum Contribution |
|-------------|-----------------|-----------------|
| 1 | \\((0, 1)\) or \\((1, 0)\) | ✓ MMT used |
| **0** | \\((0, 0)\) or \\((1, 1)\) | **✓ BJMM new** |

Since \\(\omega \ll n\), there are far more 0-bits than 1-bits in the error vector. Allowing 0-bit splitting dramatically increases the number of representations per solution.

### Overlapping Index Sets

BJMM chooses \\(|I_1| = |I_2| = p/2 + \varepsilon\), \\(|I_1 \cap I_2| = \varepsilon\). The solution is recovered via **symmetric difference**:

$$I = I_1 \;\Delta\; I_2 = (I_1 \cup I_2) \setminus (I_1 \cap I_2).$$

Elements in \\(I_1 \cap I_2\) appear on both sides, contributing \\(1+1=0\), canceling in column summation.

First-layer representation count:

$$R_1(p, \ell; \varepsilon_1) = \binom{p}{p/2} \cdot \binom{k+\ell-p}{\varepsilon_1}.$$

Adds factor \\(\binom{k+\ell-p}{\varepsilon_1}\) over MMT.

Two-layer representation count:

$$R_2(p, \ell; \varepsilon_1, \varepsilon_2) = \binom{p}{p/2} \cdot \binom{k+\ell-p}{\varepsilon_1} \cdot \binom{p/2+\varepsilon_1}{p/4+\varepsilon_2/2} \cdot \binom{k+\ell-p/2-\varepsilon_1}{\varepsilon_2}.$$

## 5. MERGE-JOIN: The Core Operation

**MERGE-JOIN**\\((L_1, L_2, r, p, t)\) returns:

$$L = \{ x + y \mid x \in L_1, y \in L_2,\ \mathrm{wt}(x+y) = p,\ (Q(x+y))[r] = t \}.$$

**Matching mechanism**: Sort \\(L_1\) by label \\((Qx)[r]\), sort \\(L_2\) by \\((Qy)[r] + t\), scan with dual pointers to find collisions. Filter inconsistent solutions (weight \\(\neq p\)) and duplicates.

Runtime: \\(\widetilde{O}(\max\{|L_1|, |L_2|, C\})\), \\(\mathbb{E}[C] = |L_1| \cdot |L_2| \cdot 2^{-r}\).

### MERGE-JOIN Python Implementation

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

## 6. COLUMN MATCH: Three-Layer Computation Tree

BJMM solves SMP via a three-layer MERGE-JOIN tree, structurally analogous to Wagner's generalized birthday problem.

### Parameter Setting

| Parameter | Definition |
|-----------|------------|
| \\(p_1\) | \\(p/2 + \varepsilon_1\) |
| \\(p_2\) | \\(p_1/2 + \varepsilon_2 = p/4 + \varepsilon_1/2 + \varepsilon_2\) |
| \\(r_1\) | \\(\approx \log_2 R_1\) |
| \\(r_2\) | \\(\approx \log_2 R_2\) |

### Algorithm Steps

**Step 3 (Layer 3)**: For \\(i = 1, \ldots, 4\), randomly partition \\([k+\ell] = P_{i,1} \cup P_{i,2}\), construct disjoint base lists

$$B_{i,1} = \{ y \mid \mathrm{wt}(y) = p_2/2,\ y_j = 0\ (j \in P_{i,2}) \}$$

$$B_{i,2} = \{ z \mid \mathrm{wt}(z) = p_2/2,\ z_j = 0\ (j \in P_{i,1}) \}$$

Then \\(L^{(2)}_i = \text{MERGE-JOIN}(B_{i,1}, B_{i,2}, r_2, p_2, t^{(2)}_i)\).

**Step 2 (Layer 2)**: Merge pairs

$$L^{(1)}_1 = \text{MERGE-JOIN}(L^{(2)}_1, L^{(2)}_2, r_1, p_1, t^{(1)}_1)$$

$$L^{(1)}_2 = \text{MERGE-JOIN}(L^{(2)}_3, L^{(2)}_4, r_1, p_1, t^{(1)}_2)$$

**Step 1 (Layer 1, Final)**: Merge to obtain solution

$$L = \text{MERGE-JOIN}(L^{(1)}_1, L^{(1)}_2, \ell, p, s)$$

Computation tree structure:

```
Layer 3:  B1,1  B1,2 | B2,1  B2,2 | B3,1  B3,2 | B4,1  B4,2
            ───────────────   ───────────────   ───────────────   ─────────
Layer 2:      L(2)_1            L(2)_2            L(2)_3            L(2)_4
                 ───────────────   ───────────────
Layer 1:           L(1)_1                  L(1)_2
                 ────────────────────────────────
Layer 0:                      L  →  SMP Solution
```

### COLUMN MATCH Python Implementation

```python
def column_match(Q, s, p, eps1, eps2):
    """
    COLUMN MATCH: solve SMP via BJMM.

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

## 7. Complexity Analysis

The three-layer MERGE-JOIN time complexity:

$$T(p, \ell; \varepsilon_1, \varepsilon_2) = \max\{ T_3, T_2, T_1 \} = \max\{ S_3, C_3, S_2, C_2, S_1, C_1 \}$$

where \\(S_i = \binom{k+\ell}{p_i} \cdot 2^{-r_i}\), \\(\mathbb{E}[C_i] = S_i^2 \cdot 2^{r_i - r_{i-1}}\).

Iteration success probability: \\(P^{-1} = \frac{\binom{n}{\omega}}{\binom{k+\ell}{p}\binom{n-k-\ell}{\omega-p}}\).

Optimizing parameters on random linear codes:

$$T_{\text{BJMM}} = 2^{0.04934n} \quad \text{(half-distance decoding, worst case)}$$

Storage complexity: \\(\approx 2^{0.0286n}\).

**McEliece typical parameters** (\\(R = 0.7577\), \\(D = 0.04\)):

| Metric | Half-Distance Decoding | Full Decoding |
|--------|----------------------|---------------|
| Time | \\(2^{0.0672n}\) | \\(2^{0.1019n}\) |
| Space | \\(2^{0.0586n}\) | \\(2^{0.0769n}\) |

## 8. Comparison with Prior Work

| Algorithm | \\(2^{\alpha n}\) (half-distance) | Key Technique |
|-----------|-------------------------------|---------------|
| Lee-Brickell | \\(2^{0.05752n}\) | \\(p < \omega\) |
| Stern | \\(2^{0.05564n}\) | MITM |
| Ball-collision | \\(2^{0.05559n}\) | Inexact matching |
| MMT | \\(2^{0.05364n}\) | Representation, disjoint |
| **BJMM** | **\\(2^{0.04934n}\)** | Overlap, \\(1+1=0\) |

BJMM improves over MMT by factor \\(2^{0.0043n}\), approximately \\(3\times\) at \\(n=2000\).

## 9. PROVABLE CM: Provable Variant

The heuristic analysis relies on uniformity of \\((Qe)[r]\). **PROVABLE CM** repeatedly calls COLUMN MATCH with fresh random target values, aborting when any list exceeds expected size by \\(2^{\gamma n}\) times (constant \\(\gamma > 0\)). Except for a negligible fraction of matrices \\(Q\), this variant succeeds with probability \\(> 1 - 3/e^2\).

## References

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in \\(2^{n/20}\): How \\(1+1=0\) Improves Information Set Decoding.* Eurocrypt 2012. ([ePrint](https://eprint.iacr.org/2012/026))
2. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in \\(\tilde{O}(2^{0.054n})\).* Asiacrypt 2011.
3. J. Stern. *A method for finding codewords of small weight.* Coding Theory 1989.
