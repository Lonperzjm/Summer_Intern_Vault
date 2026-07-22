---
type: method
title: FMPS（Flow Matching Posterior Sampling）
aliases: [FMPS, Flow Matching Posterior Sampling]
tags: [flow-matching, guidance, training-free, clean-estimate, energy-guidance]
status: stable
created: 2026-06-29
updated: 2026-06-29
sources: ["[[wiki/sources/songFlowMatchingPosterior2025]]"]
family: guidance
---

# FMPS（Flow Matching Posterior Sampling）

> family `guidance`：[[wiki/concepts/training-free-guidance|training-free guidance]] 的 **flow matching 版**——把 [[wiki/methods/freedom|FreeDoM]] 式 clean-estimate 引导接到只有速度场、没有 score 的 FM 上。

## 一句话总结

用 Proposition 1 把 FM 的**速度场改写成 surrogate score**，补上"FM 无显式 score"这一环，从而把 posterior-sampling 引导搬到 flow；引导项 $-r\beta_t\nabla_{x_t}D(\hat x_{0\mid t},c)$ 评在 clean 估计 $\hat x_{0\mid t}$ 上。

## 核心机制

| 组件 | 内容 |
|---|---|
| 底座 | 预训练无条件 flow matching（冻结），路径 $x_t=a_tx_0+b_t\epsilon$ |
| 桥 | $v_\theta=\frac{\dot b_t}{b_t}x_t+\beta_t\nabla\log p_t$，$\beta_t=a_t(\dot a_t-\frac{\dot b_t}{b_t}a_t)$ |
| 能量 | FreeDoM 式距离 $D(\hat x_{0\mid t},c)$（现成模型） |
| 引导 | $v_{\text{guided}}=v_\theta-r\beta_t\,g^1$，$g^1=\frac{\lVert v_\theta\rVert}{\lVert\nabla D\rVert}\nabla_{x_t}D$（归一化步长） |
| $\hat x_0$ | **gradient 版**：$x_t-tv_\theta$（准/贵，反传 $v_\theta$）｜**free 版**：$(x_t-b_tx_1)/a_t$（便宜/糙，$t=1$ 不可用） |
| 训练 | **无**（training-free） |

## 与 FreeDoM / DPS 的关系

- **同框架**：$x_t\to\hat x_0\to D(\hat x_0,c)\to\nabla_{x_t}D\to$ 引导（见 [[wiki/concepts/conditional-diffusion]] §3–§4）。
- **差异**：FreeDoM/DPS 在 diffusion（有 score）上；FMPS 在 FM（无 score）上，靠 Prop 1 补桥。
- FMPS-free 用前向反解避开 $\partial v_\theta/\partial x_t$ 反传——和 MPGD 省算力同动机。

## 关联

- 出处：[[wiki/sources/songFlowMatchingPosterior2025]]
- 概念：[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/energy-guidance]]、[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/tweedie-formula]]
- 同族：[[wiki/methods/freedom|FreeDoM]]（diffusion 版）、[[wiki/methods/dps|DPS]]
- 底座：[[wiki/methods/rectified-flow|RF]] / [[wiki/concepts/flow-matching|FM]]
- 研究意义：占掉 [[research/ideas]] energy-guidance 候选①轴（见 [[wiki/synthesis/energy-guidance-landscape]]）
