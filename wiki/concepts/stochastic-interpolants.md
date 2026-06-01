---
type: concept
title: Stochastic Interpolants
aliases: [stochastic interpolants, 随机插值, Albergo-Vanden-Eijnden]
tags: [flow-matching, diffusion-bridge, sde, ode, generative-model, foundational]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/albergoStochasticInterpolants2023]]", "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# Stochastic Interpolants

> 概念主页。源：[[wiki/sources/albergoStochasticInterpolants2023|Albergo, Boffi & Vanden-Eijnden 2023]]（已 ingest）。

## 一句话定义

把"**设计连接两分布的桥**"与"**怎么采样它**"解耦的统一框架：自由选一条插值
$$
x_t=\alpha(t)x_0+\beta(t)x_1+\gamma(t)z,\quad x_0\sim\rho_0,\ x_1\sim\rho_1,\ z\sim\mathcal N(0,I),
$$
连接**任意两分布**（有限时间精确到达两端）；其密度 $\rho(t)$ 同时满足一阶 transport equation（→ ODE）与**一族可调噪声系数 $\epsilon(t)$ 的 Fokker-Planck**（→ SDE）。velocity 与 score 都是**简单二次目标的唯一最小值**。

## 为什么它是"bridge SDE vs bridge ODE"的严格归宿

- **同一 $\rho(t)$，多种动力学**：确定 ODE 与一族 SDE 共享同一边缘、同一 velocity+score 估计，差别只在扩散系数 $\epsilon(t)$（$\epsilon{=}0$ 即 ODE）。这就是"bridge SDE（[[wiki/methods/ddbm|DDBM]]）vs bridge ODE（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]）"能摆进一张图的根本原因。
- **不依赖参考扩散**：插值**自由选**、不从 VP/VE noise→data 扩散推 ⇒ 这是"让 SDE bridge 脱离 diffusion 框架"的**已实现**版本。
- **收编既有方法**：[[wiki/concepts/score-sde|score diffusion]] / stochastic localization / denoising / [[wiki/methods/rectified-flow|rectified flow]] 都是特例；显式优化插值 → 还原 Schrödinger Bridge。

## 与 DDBM / DBIM / FM 的精确分工

| | Stochastic Interpolants | [[wiki/methods/ddbm\|DDBM]] | [[wiki/concepts/flow-matching\|FM]] |
|---|---|---|---|
| 桥怎么来 | 自由选 interpolant（不依赖扩散） | 从 VP/VE 扩散 + [[wiki/concepts/doob-h-transform\|Doob h]] 推 | 选 conditional probability path |
| 学什么 | velocity + score（二次目标） | denoising bridge score | conditional velocity |
| ODE/SDE | 两者皆可，$\epsilon$ 可调 | SDE 为主（[[wiki/methods/dbim\|DBIM]] 给确定版） | ODE 为主 |
| 关系 | **最一般框架** | SI 的"绑回扩散"特例 | SI 的 ODE 侧特例 |

## 与其他概念的关系

- 数学底座：[[wiki/concepts/fokker-planck-equation]]（可调系数族）、[[wiki/concepts/probability-flow-ode]]、[[wiki/concepts/score-matching]]
- 实例 / 下游：[[wiki/methods/rectified-flow]]、[[wiki/methods/ddbm]]、[[wiki/methods/dbim]]、[[wiki/concepts/diffusion-bridge]]

## 在 text-guided editing 中的作用

- 框架本身是**无条件**的；做 text-guided / target-aware 编辑、可控多样性（$\epsilon$ 旋钮）是其 Conclusion 点名的 future application 之一——是**应用侧的开放口**（理论侧已满）。详见 [[wiki/synthesis/bridge-sde-editing-landscape]] 与 [[research/ideas]]。

## 出处与引用

- [[wiki/sources/albergoStochasticInterpolants2023]]（原文，已 ingest）
- [[wiki/sources/zhouDenoisingDiffusionBridge2023]]（DDBM §6 把 SI 作为统一框架引用）
