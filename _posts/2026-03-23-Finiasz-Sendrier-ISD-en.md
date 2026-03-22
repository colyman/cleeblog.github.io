---
layout: article
title: "Finiasz–Sendrier ISD: Unified Security Bounds for Code-based Cryptography"
date: 2026-03-23
key: Finiasz-Sendrier-ISD-2026
lang: en
tags: [Code-Based-Cryptography, Information-Set-Decoding, McEliece, Post-Quantum, ISD]
bilingual: true
---

**tl;dr** Finiasz and Sendrier's 2009 work provides a **unified analysis framework** for Information Set Decoding (ISD) algorithms, deriving a **lower bound** that encompasses all known improvements (Stern, Canteaut-Chabaud, Bernstein-Lange-Peters, etc.). This work became the de facto standard for selecting secure parameters in code-based cryptosystems—the gap between the bound and the best known attacks is already very small.

<!-- more -->

## 1. Background: Why Do We Need ISD Lower Bounds?

Since McEliece proposed his public-key cryptosystem in 1978, its security has rested on the **computational hardness of syndrome decoding** (SDP) for general linear codes. SDP is NP-complete—but "hard" is asymptotic. To select parameters offering $2^{128}$ security, one needs a *precise* analysis.

Prior to 2009, several concrete attack implementations existed:

| Work | Year | Attack Complexity (bit operations) |
|------|------|-----------------------------------|
| Canteaut-Chabaud | 1998 | $2^{64.2}$ |
| Bernstein-Lange-Peters | 2008 | $2^{60.5}$ |
| Real attack (BLP) | 2008 | ~$2^{58}$ CPU cycles |

However, all these works analyzed *specific implementations*, lacking a **unified lower bound formula**—meaning if someone made a new optimization in the future, nobody knew how much security margin remained.

Finiasz and Sendrier's goal: give a **lower bound covering all known ISD variants**, so that cryptosystem designers can analyze once and use forever.

## 2. Problem Formalization

### 2.1 Computational Syndrome Decoding (CSD)

**Problem 1 (CSD)**  
Given $H \in \mathbb{F}_2^{r \times n}$, $s \in \mathbb{F}_2^r$, and integer $w > 0$, find $e \in \mathbb{F}_2^n$ with $\mathrm{wt}(e) \leq w$ and $e H^\top = s$.

This is the core security assumption behind McEliece and Niederreiter encryption.

### 2.2 Uniqueness of Solutions

In cryptography, CSD instances usually have **exactly one solution** (encryption scenario), but when $w$ exceeds the Gilbert-Varshamov bound, there may be **multiple solutions** (signatures and hashing). The complexity analysis differs between these two cases.

## 3. Birthday Attack: The Simplest Approach

The birthday attack splits the columns of $H = (H_1 \mid H_2)$ and uses the birthday paradox to find a collision:

$$L_1 = \{e_1 H_1^\top : e_1 \in W_{n/2, w/2}\}, \quad L_2 = \{s + e_2 H_2^\top : e_2 \in W_{n/2, w/2}\}$$

The collision probability is $\Pr_{n,w} \approx 4/\sqrt{\pi w}$. The birthday decoding work factor lower bound is:

$$\mathrm{WF}_{BA}(n, r, w) \geq \sqrt{2} \cdot L \cdot \log_2(2L), \quad L = \min\!\left(\sqrt{\binom{n}{w}}, 2^{r/2}\right)$$

While much better than brute force, the birthday attack is not optimal—Stern's ISD outperforms it in the cryptographic parameter regime.

## 4. Stern's ISD Algorithm

### 4.1 Core Idea

Stern (1989) combined **birthday paradox** with **traditional ISD**: instead of searching the full space, first perform **partial Gaussian elimination**, then do an **ℓ-bit collision search** in the reduced space.

Key innovations:
1. Randomly permute $H$ with $P$, apply **partial** Gaussian elimination:
$$UH_0P = \begin{bmatrix} I_{r-\ell} & H_1 \\ 0 & H_2 \end{bmatrix}$$
2. Split the solution $e$: put $p$ ones in the first $r-\ell$ positions (searchable via lookup tables), the remaining $w-p$ ones in the last $\ell$ positions (found via collision search)
3. Require $e_1 H_1^\top$ and $s + e_2 H_2^\top$ to match on ℓ bits

### 4.2 Classical Complexity

Stern's original work factor (on generator matrix):
$$T_{Stern} \approx \min_p 2^\ell \cdot \binom{n}{w} \cdot \binom{r-\ell}{w-p} \cdot \binom{k/2}{p/2}$$

This informed many practical implementations but lacked a **unified lower bound**.

## 5. Finiasz–Sendrier: A Unified Analysis

### 5.1 Generalized ISD Algorithm (Table 2)

The paper's key contribution is a **generalized version** of Stern's algorithm that:
1. Works on the **parity-check matrix** (not generator matrix), compatible with both Leon's probabilistic algorithm and Stern's approach
2. Uses **partial Gaussian elimination** treated as "free" in the idealized model
3. Introduces parameters $p$ and ℓ with a clear optimization procedure

Algorithm sketch:

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

Here $h_ℓ(x)$ denotes the first ℓ bits of $x$. $W_1 \subseteq W_{k+\ell, \lceil p/2 \rceil}$, $W_2 \subseteq W_{k+\ell, \lfloor p/2 \rfloor}$.

### 5.2 Key Assumptions

**Assumption I1**: For all examined pairs $(e_1, e_2)$, their sum $e_1 + e_2$ is uniformly and independently distributed in $W_{k+\ell, p}$.

**Assumption I2**: The execution cost approximates:
$$C \approx \ell \cdot N_1 + \ell \cdot N_2 + K_{w-p} \cdot N_3$$
where $K_{w-p}$ is the average cost of checking $\mathrm{wt}(s + (e_1+e_2)H'^T) = w-p$.

### 5.3 Work Factor Lower Bound (Proposition 2)

For the **unique solution** case ($\binom{n}{w} < 2^r$):

$$\boxed{\mathrm{WF}_{ISD}(n, r, w) \approx \min_p 2^\ell \cdot \min\!\left(\binom{n}{w}, 2^r\right) \cdot \xi \cdot \sqrt{\binom{r-\ell}{w-p}} \cdot \sqrt{\binom{k+\ell}{p}}}$$

where:
- $\ell \approx \log_2\!\left(K_{w-p} \cdot \sqrt{\binom{k}{p/2}}\right)$
- $\xi = 1 - e^{-1} \approx 0.63$ (optimal value at $z=1$ where $\xi(z) = 1-e^{-z}$)
- $p$ and ℓ are parameters to be optimized

For the **multiple solutions** case, a similar formula holds with $2^{r/2}$ as the main term.

### 5.4 Comparison with Stern's Original

Stern's original formula (on generator matrix):
$$T_{Stern} \approx \min_p 2^\ell \cdot \binom{n}{w} \cdot \binom{r-\ell}{w-p} \cdot \binom{k/2}{p/2}$$

The new version replaces $\binom{k/2}{p/2}$ with $\sqrt{\binom{k+\ell}{p}}$ and introduces $\xi \approx 0.63$, giving a gain of about $\xi \cdot 4\sqrt{\pi p/2}$—a modest constant improvement.

**What really matters is not the constant improvement, but that the bound encompasses all known optimization variants.** Regardless of how an implementor tunes parameters, the work factor cannot beat this bound.

## 6. Concrete Attack Results

The paper gives ISD lower bounds for standard McEliece parameters (Table 3):

| Params $(m, w)$ | Optimal $p$ | Optimal ℓ | Binary Work Factor |
|----------------|-------------|-----------|-------------------|
| (10, 50) | 4 | 22 | $2^{59.9}$ |
| (11, 32) | 6 | 33 | $2^{86.8}$ |
| (12, 41) | 10 | 54 | $2^{128.5}$ |

Params: code length $n = 2^m$, dimension $k = n - mw$, codimension $r = mw$.

For the classical McEliece parameters $(1024, 524)$ with $w = 50$ errors:
- Finiasz-Sendrier bound: $2^{59.9}$
- Best implementation (BLP): $2^{60.5}$
- Real attack (BLP 2008): ~$2^{58}$ CPU cycles

**The gap between the lower bound and the best known attack is already tiny**, leaving very little room for further optimization—breaking McEliece requires fundamentally new techniques.

## 7. Why This Bound Matters

### 7.1 A Tool for Designers

Traditional cryptanalysis gives attackers' upper bounds. Finiasz-Sendrier's work provides **designers' lower bounds**:
- The lower bound is **persistent**: even if future optimizations emerge, the bound holds unless entirely new techniques are introduced
- Designers benefit from **one analysis, lasting security**: no need to re-evaluate parameters every time a new attack implementation appears

### 7.2 GBA Coverage

The paper also analyzes Wagner's **Generalized Birthday Algorithm** (GBA), applicable to **multiple-solution instances** (CFS signatures, FSB hash). GBA's lower bound:
$$\mathrm{WF}_{GBA}(n, r, w) \geq \frac{r-a}{a} \cdot 2^{\frac{r-a}{a}}$$
where $a$ satisfies $\frac{1}{2^a}\binom{n}{w/2^a} = 2^{\frac{r-a}{a}}$.

### 7.3 Implications

1. **McEliece encryption** (unique solution) → ISD dominates, $2^{60}$ level (classical params) → larger params needed
2. **CFS signatures** (multiple solutions) → GBA is stronger → must use specially designed Goppa codes
3. **FSB hash** (multiple solutions) → GBA threatens, but constrained inputs (regular words) limit practicality

## 8. Conclusion

Finiasz-Sendrier 2009 established the foundational framework for parameter selection in code-based cryptography. By providing **unified lower bounds for ISD and GBA**, they enabled designers to:

1. **Precisely evaluate security margins**: the gap between bound and best attack = security margin
2. **Anchor security analysis permanently**: bounds don't expire with new implementations
3. **Compare systems uniformly**: McEliece, Niederreiter, CFS, FSB—all analyzable with the same tools

**tl;dr cont'd**: The paper's conclusion contains a line worth remembering: *"Solving CSD more efficiently than these bounds would require introducing new techniques, never applied to code-based cryptosystems."* This sentence underpins the confidence behind every code-based PQC standard finalized today.

## References

1. M. Finiasz, N. Sendrier. *Security Bounds for the Design of Code-based Cryptosystems*. Asiacrypt 2009. https://eprint.iacr.org/2009/414
2. J. Stern. *A method for finding codewords of small weight*. Coding Theory and Applications, 1989.
3. A. Canteaut, F. Chabaud. *A new algorithm for finding minimum-weight words in a linear code*. IEEE Trans. IT, 1998.
4. D. Bernstein, T. Lange, C. Peters. *Attacking and defending the McEliece cryptosystem*. PQCrypto 2008.
5. D. Wagner. *A generalized birthday problem*. Crypto 2002.
