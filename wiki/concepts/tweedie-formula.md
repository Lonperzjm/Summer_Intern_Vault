---
type: concept
title: Tweedie 公式（后验均值 $\hat x_0$）
aliases: [Tweedie, Tweedie's formula, Tweedie 公式, posterior mean, empirical Bayes, x0 估计]
tags: [diffusion, score-sde, guidance, sampling]
status: stable
created: 2026-06-24
updated: 2026-06-24
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
---

# Tweedie 公式（后验均值 $\hat x_0$）

> 概念页。把"带噪态 $x_t$ + score"换算成"对干净图 $x_0$ 的后验均值估计 $\hat x_0$"的公式。是 [[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/training-free-guidance]]、[[wiki/methods/freedom|FreeDoM]]、DPS 等一切 clean-estimate 引导的**地基**。

## 公式

VP/DDPM 约定，$x_t=\sqrt{\bar\alpha_t}x_0+\sqrt{1-\bar\alpha_t}\,\epsilon$：
$$\boxed{\ \hat x_0:=\mathbb E[x_0\mid x_t]=\frac{x_t+(1-\bar\alpha_t)\,\nabla_{x_t}\log p_t(x_t)}{\sqrt{\bar\alpha_t}}=\frac{x_t-\sqrt{1-\bar\alpha_t}\,\epsilon_\theta(x_t,t)}{\sqrt{\bar\alpha_t}}\ }$$
**关键**：预训练 score/$\epsilon$ 网络**直接**给出 $\hat x_0$，无需额外训练。

## 推导（一行版）

由 $\nabla_{x_t}\log p_t(x_t)=\dfrac{\int\nabla_{x_t}p(x_t\mid x_0)p(x_0)\,\mathrm dx_0}{p_t(x_t)}$ 与 $\nabla_{x_t}p(x_t\mid x_0)=-p(x_t\mid x_0)\dfrac{x_t-\sqrt{\bar\alpha_t}x_0}{1-\bar\alpha_t}$，得
$$\nabla_{x_t}\log p_t(x_t)=-\frac{x_t-\sqrt{\bar\alpha_t}\,\mathbb E[x_0\mid x_t]}{1-\bar\alpha_t}\ \Longrightarrow\ \text{上式}.$$
完整版见 [[wiki/concepts/conditional-diffusion]] §3。

## 变体

- **VE-SDE / NCSN**：$\hat x_0=x_t+\sigma_t^2\,\nabla\log p_t$。
- **Flow / Rectified Flow**：$\hat x_0=x_t-t\,v_\theta(x_t,t)$（速度场一步外推）——因 [[wiki/methods/rectified-flow|RF]] 轨迹近直线，$\hat x_0$ 更准、偏差更小（[[research/ideas]] energy-guidance 候选的 flow 论据）。

## 为什么重要

- **clean-estimate 引导的入口**：把带噪 $x_t$ 上无从评分的能量，换成"评在 $\hat x_0$ 上"——[[wiki/methods/freedom|FreeDoM]] / DPS / [[wiki/concepts/training-free-guidance|training-free guidance]] 全靠它。
- **偏差来源**：用 $\hat x_0$ 这一个点替期望 $\mathbb E[f(x_0)\mid x_t]$ 有 **Jensen gap**，高噪声（后验宽）下变大——这是该路线的核心弱点。

## 关系

- 母页：[[wiki/concepts/conditional-diffusion]]（§3–§4）；底座 [[wiki/concepts/score-sde]]
- 用它的方法：[[wiki/methods/freedom]]、[[wiki/methods/egsde]]（对照：EGSDE 刻意**不**用 $\hat x_0$，走 noisy-aligned）
- 子族：[[wiki/concepts/training-free-guidance]]
