---
type: research-note
title: 高维空间中 RF 路径交叉概率分析
tags: [rectified-flow, high-dimensional, crossing-probability, original-analysis]
status: active
created: 2026-07-27
updated: 2026-07-27
triggered_by: ["[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]"]
---

# 高维空间中 RF 路径交叉概率分析

> 原创分析。由阅读 [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]] 触发。

## 核心问题

HRF 的卖点是"让路径可交叉"。但在实际高维 latent 空间中，交叉本身的概率有多大？

## 分析

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785140429486.webp]]

如上图所见，如果在高维空间中很不幸有两条直线相交了（或者几乎要相交了），那么相交处就会有一个大的速度突变（变为两个速度的概率平均）。在离散化时，如果你很不幸跳到他们的交点附近了，再加一个雷霆大跳，就可能会 OOD。

首先考虑的是这个"不幸"的概率大不大。毕竟 HRF 的代价还是蛮大的。数学推导得知，$d \gg 1$ 维，$x_i \in [0,1]$ 的空间内随机两条直线，它们的最小距离 $d_{\min}$：

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785238406195.webp]]

$$E(d_{\min}) \propto \sqrt{d}, \quad D(d_{\min}) \propto 1$$

$d_{\min}$ 足够多满足正态分布。如下图所示，可以发现如果 $d$ 很大，比如说为 $20$ 万，相交，即便是几乎要相交的概率，其实也不大。

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785207600516.webp]]
![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785147289964.webp]]

## 结论

HRF 几乎一倍的输入量增加是不经济的。高维空间中路径交叉本身就是低概率事件，不值得为此付出翻倍参数量的代价。

## 后续延伸

→ [[research/notes/2026-07-28-singularity-unified-framework|奇异点统一框架]]（将此分析与 FDS 结合）
