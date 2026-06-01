---
type: concept
title: 扩散桥（diffusion bridge）
aliases: [diffusion bridge, 扩散桥, bridge process, 桥过程, Brownian bridge]
tags: [sde, diffusion-bridge, image-translation, generative-model]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# 扩散桥（diffusion bridge）

> 概念页。源：[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]]。

## 一句话定义

一族在**两个给定端点之间插值**的随机过程：$x_0\to x_t\to x_T$ 两端固定（$x_0\sim$ data、$x_T=y$ 是 condition），中间是**随机**扩散路径（不是确定性直线）。对 VE/VP 高斯核，中间点可写成
$$
x_t=a_t x_T+b_t x_0+\sqrt{c_t}\,\epsilon,\qquad \epsilon\sim\mathcal N(0,I),
$$
系数由所选扩散过程 + [[wiki/concepts/doob-h-transform|Doob's h-transform]] 决定（[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] 公式 8）。Brownian bridge 是 VE 桥的特例。

## 为什么需要"桥"而不是"流"

普通扩散是 data↔noise 的单端约束；桥是 **data↔data（或 data↔condition）的双端约束**，天然适配 image translation / restoration / editing 这类"输入不是噪声"的任务。中间路径随机（而非确定直线）保证了反向采样的多样性，避免 ODE 平均化导致的模糊（DDBM 纯 ODE 采样会糊，需注入 stochasticity）。

## bridge SDE vs bridge ODE（与 flow 的精确分界）

桥可以用**随机 SDE** 或**确定 ODE** 两种动力学实现同一条边际路径——这是「DDBM=bridge SDE，flow-matching/RF=bridge ODE」这一 lens 的核心。三处不可消差异：

| | bridge SDE（[[wiki/methods/ddbm\|DDBM]]） | bridge ODE（[[wiki/methods/rectified-flow\|RF]] / [[wiki/concepts/flow-matching\|FM]]） |
|---|---|---|
| 动力学 | 随机（score-based） | 确定（velocity field） |
| 端点 | [[wiki/concepts/doob-h-transform\|Doob h-transform]] 钉死 | 直接指定插值（常直线） |
| 耦合 | paired $(x_0,x_T)$ | 通常 independent / OT |
| 学什么 | score $\nabla\log q(x_t\mid x_T)$ | 速度场 $v$ |

> ⚠️ SDE↔ODE 可互转（PF-ODE / 加噪），所以这不是两种方法的**本质**分界，而是 default 实现 + 学习对象的差异。严格统一框架见 [[wiki/concepts/stochastic-interpolants]]。

## 相关工作谱系（DDBM §6）

- **Schrödinger Bridge（SB）**：De Bortoli et al. 2021（IPF）、Liu et al. 2023（tractable SB = I²SB）、Bridge-Matching（Shi et al. 2023，Iterative Markovian Fitting）
- **Doob h-transform 桥**：Liu et al. 2022b、Somnath et al. 2023、Peluchetti 2023、Heng et al. 2021
- **Brownian Bridge 直接迭代**：Delbracio & Milanfar 2023（图像复原）、Li et al. 2023
- DDBM 的差异：从**任意 VP/VE 连续时间扩散**构造桥（Brownian bridge 只是 VE 特例），并复用扩散设计

## 与其他概念的关系

- 构造工具：[[wiki/concepts/doob-h-transform]] / [[wiki/concepts/infinitesimal-generator]]
- 实现方法：[[wiki/methods/ddbm]]
- 对照：[[wiki/concepts/diffusion-process]]（单端）、[[wiki/concepts/flow-matching]] / [[wiki/methods/rectified-flow]]（bridge ODE）、[[wiki/concepts/stochastic-interpolants]]（统一框架）

## 出处与引用

- [[wiki/sources/zhouDenoisingDiffusionBridge2023]]（DDBM）
- Särkkä & Solin 2019（Applied Stochastic Differential Equations）
