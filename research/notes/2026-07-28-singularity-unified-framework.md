---
type: research-note
title: 奇异点统一框架：Spatial vs Temporal 修正策略
tags: [flow-matching, singularity, FDS, adaptive-step, original-analysis]
status: active
created: 2026-07-28
updated: 2026-07-28
triggered_by: ["[[wiki/sources/chaTrainingFreeRefinementFlow2026]]", "[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]"]
---

# 奇异点统一框架：Spatial vs Temporal 修正策略

> 原创分析。由阅读 [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]] + [[research/notes/2026-07-27-high-dim-crossing-probability|高维交叉概率分析]] 触发。

## 设定

如 [[research/notes/2026-07-27-high-dim-crossing-probability]] 所言，我们的高维线段很少相交。三相交应该更少才对。故只考虑双相交线段。我们称相交处为奇异点，或者多模态点。

直线最小的距离为 $d(t)_{\min}$，离散步长为 $\Delta = v\delta t$，模型的表达能力为 $l$，代表模型不能良好拟合超过此尺度的转向。

![[chaTrainingFreeRefinementFlow2026plus-1785231261637.webp]]

## 关键不等式

可以估计 $\Delta \approx 100l$，即模型的表达能力远远好于采样步长的分辨率。

再估计 $E(d_{\min})$。当 $d_{\min} < l$ 时模型不能良好拟合速度场，当 $d_{\min} < \Delta$ 时采样点可能落入 OOD。

![[raw/assets/chaTrainingFreeRefinementFlow2026plus-1785241803614.webp]]

$$l \ll \Delta \ll E(d_{\min})$$

- $l \approx \Delta / 100$（网络可以拟合极尖锐的转向）
- $E(d_{\min}) \approx 10\text{-}40 \times \Delta$（多数点远离奇异点，不受影响）

## 结论

问题不在"模型能否拟合奇异点附近的速度场"（能），而在"离散化 step 是否恰好跨过奇异点导致 OOD"。低概率但一旦命中后果严重。

## 两种修正策略

直观的解决方法有两个：

![[chaTrainingFreeRefinementFlow2026plus-1785241855762.webp]]

1. **缩小步长**（temporal）：检测到 $v$ 变化，缩小步长，直到 $v$ 变化缩小

![[chaTrainingFreeRefinementFlow2026plus-1785241874885.webp]]

2. **增大步长**（temporal）：检测到 $v$ 变化，增大步长，直到 $v$ 变化缩小（一步跨过奇异区）

| 策略 | 方向 | 做法 | 代表 |
|------|------|------|------|
| 缩小步长 | temporal | 检测 $v$ 变化 → 减小 $\delta t$，精细通过 | adaptive step-size solvers |
| 增大步长 | temporal | 检测 $v$ 变化 → 增大 $\delta t$，跨过去 | （新思路，待验证） |

注：FDS 走了另一条路（spatial shift），与上述 temporal 策略正交。

## 开放问题

- [ ] "增大步长跨过去"在 ODE 精度上有代价——是否可以跨过后做 corrector step 补偿？
- [ ] 能否用 divergence 信号同时指导 adaptive step-size？（高 divergence → 决定缩小 or 增大步长）
- [ ] 两种 temporal 策略与 FDS 的 spatial 策略组合效果如何？toy experiment 验证优先级

## 前序

← [[research/notes/2026-07-27-high-dim-crossing-probability|高维交叉概率分析]]
