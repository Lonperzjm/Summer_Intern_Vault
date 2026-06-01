---
type: concept
title: Stochastic Interpolants
aliases: [stochastic interpolants, 随机插值, Albergo-Vanden-Eijnden]
tags: [flow-matching, diffusion-bridge, sde, ode, generative-model]
status: draft
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# Stochastic Interpolants

> 概念页（**draft**：原文 Albergo & Vanden-Eijnden 2023 尚未 ingest，本页据 [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] §6 的转述 + 本 vault 多次讨论先立枢纽，待原文 ingest 后补全）。

## 为什么单立此页

它是「**bridge SDE（[[wiki/methods/ddbm|DDBM]]）vs bridge ODE（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]）**」这一对照的**严格数学归宿**——也是 [[wiki/concepts/flow-matching|FM]]、[[wiki/concepts/score-sde|score diffusion]]、[[wiki/concepts/diffusion-bridge|扩散桥]] 三条线唯一干净的公约数。多篇 source 反复把它标为"待 ingest"，故先建页承接反链。

## 一句话（据 DDBM §6 转述）

固定一条连接两端分布的 **interpolant** $x_t=I(t,x_0,x_1)$（+ 可选噪声），则同一条边际路径既能用 **ODE** 实现、也能用 **SDE** 实现（diffusion 系数是自由旋钮：取 0 得确定 ODE，取正得随机 SDE）。Albergo et al. 2023 由此给出"**统一 flow 与 diffusion**"的一般理论，并明确**一座桥可从 ODE 与 SDE 两个视角同时构造**，且**不需要 Doob's h-function**（与 [[wiki/concepts/doob-h-transform|DDBM 的 h-transform 路线]]不同）。

## 与 DDBM / FM / RF 的关系（核心 takeaway）

- 「DDBM = bridge SDE，flow = bridge ODE」之所以**不是两个孤立方法**而能摆到一张图上，正是因为 stochastic interpolants 把它们都纳为"固定 interpolant + 选 diffusion 系数"的实例。
- 但**共享框架 ≠ 互为特例**：DDBM 用的是 **denoising bridge score-matching loss**（学 score），FM/RF 学 velocity field；DDBM 自承"uses a different ... loss than this class of models"。
- 因此 DDBM 论文标题里的"unifies OT-Flow-Matching"应理解为**同框架叙事**（经 stochastic interpolants）+ 有条件极限约化（§6.1 Case 2，noiseless $c\to0$ + VE schedule），而非严格包含。

## 待补（ingest 原文后）

- [ ] interpolant $I(t,x_0,x_1)$ 的形式族与对噪声项的处理
- [ ] 从 interpolant 同时导出 ODE 速度场与 SDE score 的恒等式
- [ ] 与 [[wiki/concepts/conditional-flow-matching|CFM]]、[[wiki/concepts/reflow|reflow]] 的精确关系
- [ ] SD3 / FLUX 等工业实现是否走 interpolant 视角

## 出处与引用

- [[wiki/sources/zhouDenoisingDiffusionBridge2023]]（DDBM §6 对 stochastic interpolants 的转述）
- Albergo & Vanden-Eijnden 2023；Albergo et al. 2023（**原文待 ingest**）
