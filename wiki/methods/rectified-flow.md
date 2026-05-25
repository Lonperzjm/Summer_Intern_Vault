---
type: method
title: Rectified Flow
aliases: [Rectified Flow, rectified flow, reflow, "Liu et al. 2022"]
tags: [flow-matching, ode, rectified-flow, sampling-acceleration]
status: draft
created: 2026-05-24
updated: 2026-05-24
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
family: other
---

# Rectified Flow

> **Stub**：尚未 ingest 原文（Liu et al. 2022, *Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow*, ICLR 2023）。当前内容据 [[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 的并行工作交叉引用整理，待 ingest 后扩充。

## 一句话

学一条把噪声分布"尽可能走直线"运输到数据分布的 ODE；再用 **reflow**（rectification）迭代把轨迹拉直，逼近直线后可少步乃至**单步**生成。与 [[wiki/concepts/flow-matching|Flow Matching]] 并行/同源——其线性插值 $x_t=(1-t)x_0+t x_1$ 本质就是 FM 的 [[wiki/concepts/optimal-transport-path|OT 路径]]（去掉 $\sigma_{\min}$ 项的边界处理）。

## 为什么重要（对本 wiki）

- 是 [[wiki/concepts/flow-matching|FM]] → 工业级文生图的桥：**SD3 / FLUX 的训练目标**即 rectified-flow 一系。
- **reflow** 提供"训练后再把 ODE 拉直"的加速思路，补强 [[wiki/overview]] 推论 3——采样加速可来自**路径/轨迹设计**，而非只靠采样器或蒸馏。
- 落点：overview「主要派系 → flow-matching-based」的 text-guided editing 方法多半建立在 RF 模型（SD3 / FLUX）之上（如 RF-Inversion）。

## 关系

- 同源 / 并行：[[wiki/concepts/flow-matching]]、[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/optimal-transport-path]]
- 采样近亲：[[wiki/concepts/probability-flow-ode]]（都是确定性 ODE 生成；RF 额外做轨迹拉直）
- 对照：[[wiki/methods/ddim]]（diffusion 的训练 + flow 的采样）vs RF（连训练也 flow 化 + 拉直）
- 下游模型（待 ingest）：SD3（Esser et al. 2024）、FLUX

## 待补

- [ ] ingest Liu et al. 2022 原文：reflow 精确流程、k-rectification、1-step 蒸馏与质量损失
- [ ] 与 FM OT 路径的精确异同（边界条件、$\sigma_{\min}$、是否做 reflow）
- 出处：待 ingest 原文（暂引自 [[wiki/sources/lipmanFlowMatchingGenerative2023]] 的并行工作讨论）
