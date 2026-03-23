---
layout: article
title: "BJMM Algorithm: How 1+1=0 Improves Information Set Decoding"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: en
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD, McEliece, BJMM, 2012]
bilingual: true
---

> **tl;dr**: BJMM (Eurocrypt 2012) reduces the complexity of decoding random binary linear codes from MMT's $2^{0.05364n}$ to $2^{0.04934n}$. The key innovation is **allowing index sets to overlap** ($|I_1 \cap I_2| = \varepsilon$), leveraging the fact that $1+1=0$ in $\mathbb{F}_2$ to split both 1-positions *and* 0-positions, dramatically increasing the number of representations per solution.

<!-- more -->

## 1. Background: Information Set Decoding

The syndrome decoding problem is central to code-based cryptography. Given a parity-check matrix $H \in \mathbb{F}_2^{(n-k)\times n}$ and syndrome $s = He$ for an unknown error vector $e$ of weight $\omega$, the goal is to recover $e$.

**Information Set Decoding (ISD)**, introduced by Prange (1962), is the most fundamental generic approach. The key idea: permute and Gaussian-eliminate $H$ to quasi-systematic form $(Q \mid I_{n-k})$, then search for a truncated error vector $\tilde{e}_1 \in \mathbb{F}_2^k$ with exactly $p$ ones, which determines the full error vector.

Successive improvements refined this framework:
- **Lee-Brickell (1988)**: exploit $p < \omega$ → $2^{0.05752n}$
- **Stern (1989)**: Meet-in-the-Middle (MITM) → $2^{0.05564n}$
- **Ball-collision** (Bernstein, Lange & Peters, CRYPTO 2011): non-exact matching → $2^{0.05559n}$
- **MMT** (May, Meurer & Thomae, Asiacrypt 2011): representation technique → $2^{0.05364n}$

## 2. Submatrix Matching Problem (SMP)

MMT reformulated the ISD search phase as the **Submatrix Matching Problem (SMP)**:

> Given a random matrix $Q \in \mathbb{F}_2^{l \times (k+l)}$ and target vector $s \in \mathbb{F}_2^l$, find $I \subseteq [1, k+l]$, $|I|=p$, such that
> 
> $$\sigma(Q_I) := \sum_{i \in I} q_i = s,$$
> 
> where $q_i$ is the $i$-th column of $Q$.

MMT's insight: each solution $I$ can be written as $I = I_1 \cup I_2$ with $|I_1| = |I_2| = p/2$, $I_1 \cap I_2 = \varnothing$. This gives each solution $\binom{p}{p/2}$ representations, allowing lists to be shrunk by that factor.

## 3. The Key Insight: $1+1=0$

MMT's limitation: $I_1$ and $I_2$ must be **disjoint**. Each position in $I$ can appear in either $I_1$ or $I_2$, enabling splits like:

$$
1 = 1+0 \quad \text{or} \quad 1 = 0+1.
$$

**BJMM's breakthrough**: relax the disjointness constraint, allowing $|I_1 \cap I_2| = \varepsilon > 0$. The solution is now the **symmetric difference**:

$$
I = I_1 \,\Delta\, I_2 = (I_1 \cup I_2) \setminus (I_1 \cap I_2).
$$

This unlocks a critical advantage: **0-positions can also be split!**

- A 0-position not in $I$ may appear in both $I_1$ and $I_2$ (i.e., $(1,1)$). Since $1+1=0$ in $\mathbb{F}_2$, it contributes nothing to the symmetric difference.
- Or it may appear in neither (i.e., $(0,0)$), also contributing nothing.

> **Representation count comparison**:
> - MMT: $\displaystyle \binom{p}{p/2}$ representations per solution
> - BJMM: $\displaystyle \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon}$ representations (multiplied by $\binom{k+l-p}{\varepsilon}$)

Since $k+l-p \gg p$ (a codeword has far more zeros than ones), the extra binomial factor is substantial — this is where the speedup comes from.

## 4. COLUMN MATCH: A Three-Layer Algorithm

BJMM implements the extended representation idea via a **three-layer divide-and-conquer algorithm**, structurally similar to Wagner's generalized birthday algorithm.

### 4.1 Parameters

$$
p_1 = \frac{p}{2} + \varepsilon_1, \quad
p_2 = \frac{p_1}{2} + \varepsilon_2 = \frac{p}{4} + \frac{\varepsilon_1}{2} + \varepsilon_2.
$$

The number of representations at each layer:

$$
R_1 = \binom{p}{p/2} \cdot \binom{k+l-p}{\varepsilon_1}, \quad
R_2 = \binom{p_1}{p_1/2} \cdot \binom{k+l-p_1}{\varepsilon_2}.
$$

### 4.2 MERGE-JOIN: The Building Block

The **MERGE-JOIN** primitive takes two lists $L_1, L_2$ of length-$(k+l)$ binary vectors and finds all pairs $(x, y) \in L_1 \times L_2$ such that:

1. $\mathrm{wt}(x + y) = p$ (weight constraint)
2. $(Q(x+y))[r] = t$ (first $r$ bits match target $t$)

Implemented via sorted lists + two-pointer collision detection (Knuth's algorithm), with complexity $O(\max\{|L_1|, |L_2|, C\})$, where $C \approx |L_1||L_2|/2^r$.

### 4.3 Three-Layer Structure

```
Layer 3 (bottom): 4 pairs of base lists Bi,1, Bi,2  (disjoint splits)
Layer 2:           L(2)_i = MERGE-JOIN(Bi,1, Bi,2, r2, p2, t(2)_i)  (i = 1..4)
Layer 1:           L(1)_j = MERGE-JOIN(L(2)_{2j-1}, L(2)_{2j}, r1, p1, t(1)_j)  (j = 1,2)
Layer 0 (top):     L       = MERGE-JOIN(L(1)_1, L(1)_2, l, p, s)  →  solution
```

**Layer 3 — Base lists**: For $L^{(2)}_1$, partition $[k+l]$ randomly into equal halves $P_1, P_2$, and construct disjoint lists:

$$
B_1 = \{ y \mid \mathrm{wt}(y) = p_2/2,\; y_i = 0 \; \forall i \in P_2 \}, \quad
B_2 = \{ z \mid \mathrm{wt}(z) = p_2/2,\; z_i = 0 \; \forall i \in P_1 \}.
$$

Then $L^{(2)}_1 = \text{MERGE-JOIN}(B_1, B_2, r_2, p_2, t^{(2)}_1)$.

**Layer 2**: Each $L^{(2)}_i$ contains vectors of weight $p_2$, representing half of the layer-1 representation.

**Layer 1**: MERGE-JOIN on $(L^{(2)}_1, L^{(2)}_2)$ and $(L^{(2)}_3, L^{(2)}_4)$ produces $L^{(1)}_1, L^{(1)}_2$ of weight $p_1$.

**Layer 0**: Final MERGE-JOIN on $(L^{(1)}_1, L^{(1)}_2)$ with full matching on $l$ coordinates outputs SMP solutions.

### 4.4 Correctness

By the symmetric difference property: for $|I_1| = |I_2| = p_1 + \varepsilon_1$ with $|I_1 \cap I_2| = \varepsilon_1$,

$$
|I_1 \Delta I_2| = |I_1| + |I_2| - 2|I_1 \cap I_2| = 2(p_1 - \varepsilon_1) = p.
$$

Each MERGE-JOIN layer preserves the weight invariant, gradually collapsing representations back to valid solutions of the original SMP.

## 5. Complexity Analysis

### 5.1 List Sizes

Base list size: $S_3 = \binom{(k+l)/2}{p_2/2}$. Assuming uniform distribution of partial sums:

$$
S_2 = \binom{k+l}{p_2} \cdot 2^{-r_2}, \quad S_1 = \binom{k+l}{p_1} \cdot 2^{-r_1},
$$

where $r_i \approx \log_2 R_i$ are the constrained coordinate counts. Total time:

$$
T = \max\{T_3, T_2, T_1\}.
$$

### 5.2 Overall ISD Complexity

Expected running time:

$$
T_{\text{ISD}} = P^{-1} \cdot T(p, l; \varepsilon_1, \varepsilon_2),
$$

with iteration probability $P = \frac{\binom{k+l}{p}\binom{n-k-l}{\omega-p}}{\binom{n}{\omega}}$.

Optimal parameters give:

| Algorithm | Half-distance decoding | Space coefficient |
|-----------|------------------------|-------------------|
| Lee-Brickell | $2^{0.05752n}$ | — |
| Stern | $2^{0.05564n}$ | $2^{0.0135n}$ |
| Ball-collision | $2^{0.05559n}$ | $2^{0.0148n}$ |
| MMT | $2^{0.05364n}$ | $2^{0.0216n}$ |
| **BJMM** | $2^{0.04934n}$ | $2^{0.0286n}$ |

For typical McEliece parameters ($D=0.04$, $R=0.7577$), BJMM achieves $2^{0.0672n}$ vs. MMT's $2^{0.0760n}$.

## 6. Conclusion

BJMM's core contribution is **extending the representation technique** by allowing index set overlap, cleverly exploiting $1+1=0$ in $\mathbb{F}_2$ to split zero-positions alongside one-positions. The insight appears almost obvious in hindsight ("we only split 1's before, but 0's can also split"), yet the practical impact is significant — reducing decoding complexity by an exponential factor of $2^{0.0043n}$.

While later sieving-based ISD algorithms pushed complexity even lower, BJMM's three-way decomposition framework and the $1+1=0$ insight remain essential reading for understanding the evolution of modern ISD algorithms.

## References

1. A. Becker, A. Joux, A. May, A. Meurer. *Decoding Random Binary Linear Codes in $2^{n/20}$: How $1+1=0$ Improves Information Set Decoding*. Eurocrypt 2012. ([ePrint 2012/026](https://eprint.iacr.org/2012/026))
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. May, A. Meurer, E. Thomae. *Decoding Random Linear Codes in $\tilde{O}(2^{0.054n})$*. Asiacrypt 2011.
4. D. J. Bernstein, T. Lange, C. Peters. *Smaller Decoding Exponents: Ball-Collision Decoding*. Crypto 2011.
