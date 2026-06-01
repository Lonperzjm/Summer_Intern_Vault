---
type: concept
title: 生成元（infinitesimal generator）与 Kolmogorov backward / Fokker-Planck / h-transform
aliases: [infinitesimal generator, 生成元, generator, Kolmogorov backward equation, 后向方程]
tags: [sde, stochastic-process, diffusion, doob-h-transform]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# 生成元（infinitesimal generator）

> 概念页。数学骨架来自用户手写推导 [[raw/notes/生成元方法对于SDE]]，对应 [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] §4「生成元方法」。本页解释：生成元是什么、它怎么同时给出 Kolmogorov backward、[[wiki/concepts/fokker-planck-equation|Fokker-Planck]] 与 [[wiki/concepts/doob-h-transform|Doob's h-transform]] 后的漂移。

## 一句话定义

生成元 $\mathcal L_t$ 是把"随机过程的**局部运动规则**"翻译成"可对函数/分布做微积分的算子"的工具。对 Itô SDE $\mathrm dX_t=f\,\mathrm dt+g\,\mathrm dW_t$：
$$
\mathcal L_t\phi(x)=\lim_{\Delta t\to0}\frac{\mathbb E[\phi(X_{t+\Delta t})\mid X_t=x]-\phi(x)}{\Delta t}
=f\cdot\nabla\phi+\tfrac12 g(t)^2\Delta\phi.
$$
它跟踪"在 $x$ 处的粒子，$\Delta t$ 后测试函数 $\phi$ 的期望变化率"。

## 三件由它派生的事

### 1. Kolmogorov backward equation

令 $h(x,t)=p(X_T=y\mid X_t=x)$（"从 $x$ 出发未来到 $y$ 的可能性"）。由 $\mathbb E[h(X_{t+\Delta t},t+\Delta t)\mid X_t=x]=h(x,t)$（鞅性）展开得
$$
\partial_t h+\mathcal L_t h=0.
$$

### 2. Fokker-Planck = 生成元的伴随

对密度 $p_t$ 用 $\int p_t\,\mathcal L_t\phi=\int\phi\,\mathcal L_t^* p_t$ 定义伴随算子，得 $\partial_t p_t=\mathcal L_t^* p_t$，分部积分给出
$$
\frac{\partial p_t}{\partial t}=-\nabla\!\cdot(f p_t)+\tfrac12 g^2\Delta p_t,
$$
即 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]]（前向 Kolmogorov）。**backward 作用在"观测函数"上、forward（FP）作用在"密度"上，两者是同一生成元的本体与伴随。**

### 3. Doob's h-transform 的生成元

条件化转移算子 $P^h_{s,t}\phi=\tfrac1{h(x,s)}P_{s,t}(h\phi)$，配合 $\partial_t h+\mathcal L_t h=0$ 求瞬时变化率：
$$
\mathcal L_t^h\phi=\tfrac1h\mathcal L_t(h\phi)-\tfrac{\phi}{h}\mathcal L_t h
=\big[f+g^2\nabla\log h\big]\cdot\nabla\phi+\tfrac12 g^2\Delta\phi.
$$
读出新 SDE 漂移多了一项 $g^2\nabla\log h$：
$$
\mathrm dX_t=\big[f(X_t,t)+g(t)^2\nabla_x\log h(X_t,t)\big]\mathrm dt+g(t)\,\mathrm dW_t.
$$
这正是 [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] forward bridge SDE（公式 5）的 $h$ 项来源。详见 [[wiki/concepts/doob-h-transform]]。

## 与其他概念的关系

- 伴随给出 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]]；backward 给出 h 的演化方程
- 是 [[wiki/concepts/doob-h-transform|Doob's h-transform]] 漂移 $g^2\nabla\log h$ 的推导引擎
- 与 [[wiki/concepts/score-sde|Score SDE]] 的反向 SDE 同属"用算子/伴随刻画分布演化"的工具箱（Anderson 1982 的 time reversal 亦可由此读出）

## 出处与引用

- [[raw/notes/生成元方法对于SDE]]（用户手写推导，本页骨架）
- [[wiki/sources/zhouDenoisingDiffusionBridge2023]]（DDBM §4 生成元方法）
- 经典：Øksendal《Stochastic Differential Equations》；Särkkä & Solin 2019
