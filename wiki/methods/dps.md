---
type: method
title: DPS（Diffusion Posterior Sampling）
aliases: [DPS, Diffusion Posterior Sampling, "Chung et al. 2023"]
tags: [diffusion, guidance, training-free, clean-estimate, inverse-problems]
status: stable
created: 2026-06-24
updated: 2026-07-24
sources: []
---

# DPS（Diffusion Posterior Sampling）

> ⚠️ **draft / 待 ingest 原文**（Chung et al., ICLR 2023, [arXiv 2209.14687](https://arxiv.org/abs/2209.14687)）。本页据多轮讨论与 [[wiki/concepts/training-free-guidance]] 框架先建骨架，正文细节/数字待读原文补。
> [[wiki/methods/freedom|FreeDoM]] 的**最近邻**：同是 Tweedie $\hat x_0$ + 点估计引导，DPS 原用于反问题、FreeDoM 推广到任意现成条件能量。

## 一句话总结

求解逆问题（去模糊/补全/相位恢复）的训练-free 引导：用 Tweedie 估 $\hat x_0$，把难算的似然 $p(y\mid x_t)$ 近似成 $p(y\mid\hat x_0)$，梯度 $\nabla_{x_t}\log p(y\mid\hat x_0)$ 当引导。

## 核心机制

- **后验采样目标**：$\nabla_{x_t}\log p_t(x_t\mid y)=\nabla\log p_t(x_t)+\nabla_{x_t}\log p(y\mid x_t)$（见 [[wiki/concepts/conditional-diffusion]] §2）。
- **关键近似（点估计）**：$p(y\mid x_t)=\mathbb E_{p(x_0\mid x_t)}[p(y\mid x_0)]\approx p(y\mid\hat x_0)$，$\hat x_0$ 由 [[wiki/concepts/tweedie-formula|Tweedie]] 给出。
- **引导**：$\nabla_{x_t}\log p(y\mid\hat x_0(x_t))=\big(\partial\hat x_0/\partial x_t\big)^\top\nabla_{\hat x_0}\log p(y\mid\hat x_0)$——反传过 score 网络。对线性逆问题 $y=Ax_0+n$，能量取 $\lVert y-A\hat x_0\rVert^2$。
- **步长**：启发式 $\rho_t\propto 1/\lVert y-A\hat x_0\rVert$。

## 关系

- 概念：[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/tweedie-formula]]、[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/energy-guidance]]
- 最近邻 / 推广：[[wiki/methods/freedom|FreeDoM]]（任意现成条件能量）；统一框架 TFG（见 [[wiki/concepts/training-free-guidance]]）
- 对照：[[wiki/methods/egsde|EGSDE]]（noisy-aligned，相反取舍）
- 待补：ingest 原文（噪声-感知步长、measurement consistency、各逆问题数字）
