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

直线最小的距离为 $d(t)_{\min}$，离散步长为 $\Delta = v\delta t$，模型的空间分辨率为 $l$，定义为：当两条路线距离 $d_{\min} < l$ 时，模型不能良好拟合分离的路线，$v$ 会在该尺度下被平均（退化为多模态的期望）。

![[chaTrainingFreeRefinementFlow2026plus-1785231261637.webp]]

## 关键关系

$E(d_{\min})$ 的均值约为 $\Delta$ 的 10–40 倍（多数采样点远离奇异区）。

两个临界条件（需同时满足才产生 OOD）：
1. $d_{\min} < l$：两条路线距离足够近，速度场退化为平均
2. 采样时离散点恰好落入该平均区域内

![[raw/assets/chaTrainingFreeRefinementFlow2026plus-1785241803614.webp]]

**待验证假设**：$\Delta \gg l$（即采样步长远大于模型的空间分辨率），但 $l$ 的具体量级目前无法确定，需要实验测量。

若该假设成立，含义是：采样点从较远处一步就能跳入平均区（因为 $\Delta$ 大），而平均区本身很小（$l$ 小）——小陷阱 + 大步长 = 容易踩进去但难以预见。

## 结论

OOD 机制：存在路径近交叉区域（$d_{\min} < l$，速度场退化为平均）→ 采样点因离散步长 $\Delta$ 足够大，一步跳入该平均区域 → 拿到不属于任何真实路径的平均速度 → 被弹射到无人区。

这是低概率事件（高维空间中近交叉本身少，且还需要采样点恰好踩进去），但一旦命中后果严重。

**开放问题**：$l$ 的量级（即平均区域有多大）目前未知，需要实验测量。这直接决定了"踩进去"的概率有多高。

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

注：FDS 走了另一条路（spatial shift），与上述 temporal 策略正交——但这需要进一步确认是否属于同一问题的解。

## 开放问题

- [ ] $l$ 的量级如何实验测量？（候选：在 toy 2D setting 中，逐步缩小两条路线间距，观察模型预测速度何时开始退化为平均）
- [ ] "增大步长跨过去"在 ODE 精度上有代价——是否可以跨过后做 corrector step 补偿？
- [ ] 能否用 divergence 信号同时指导 adaptive step-size？（高 divergence → 决定缩小 or 增大步长）
- [ ] 两种 temporal 策略与 FDS 的 spatial 策略是否真正正交？需要更严格的论证

## 前序

← [[research/notes/2026-07-27-high-dim-crossing-probability|高维交叉概率分析]]
