---
layout: article
title: "BJMM Algorithm"
date: 2026-03-24
key: BJMM-Algorithm-2026
lang: zh
tags: [Code-Based-Cryptography, Information-Set-Decoding, ISD]
hidden: true
---

**tl;dr**: BJMM (Eurocrypt 2012) improves ISD complexity to 2^{0.04934n} using overlapping index sets.

<!-- more -->

## Background

BJMM (Becker, Joux, May, Meurer, Eurocrypt 2012) improves information set decoding through a three-way decomposition technique.

## Key Innovation

Allows index sets to overlap, using the 1+1=0 property in GF(2) to split zero positions.

## Three-Layer Algorithm

COLUMN MATCH uses three MERGE-JOIN layers to find solutions efficiently.

## Complexity

| Algorithm | Complexity |
|-----------|------------|
| Lee-Brickell | 2^{0.05752n} |
| Stern | 2^{0.05564n} |
| MMT | 2^{0.05364n} |
| BJMM | 2^{0.04934n} |

## References

1. Becker, Joux, May, Meurer. Eurocrypt 2012.
