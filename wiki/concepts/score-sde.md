---
type: concept
title: Score SDE（连续时间 score-based 生成框架）
aliases: [Score SDE, "Yang Song et al. 2021", score-based SDE, 连续时间极限]
tags: [diffusion, score-based, sde]
status: draft
created: 2026-05-14
updated: 2026-05-14
sources: ["[[wiki/sources/songDenoisingDiffusionImplicit2022]]"]
---

# Score SDE（连续时间 score-based 生成框架）

> Stub 页。Yang Song et al., *Score-Based Generative Modeling through Stochastic Differential Equations*, ICLR 2021。等 ingest 原始文献后扩充。
> 注：此处 "Song" 指 **Yang Song**；与 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 的 "Song"（**Jiaming Song**）不是同一人，二者均为 ICLR 2021。

## 一句话定义

把离散的加噪 / 去噪链取**连续时间极限**，写成一对随机微分方程：前向 SDE 把数据扩散成噪声，对应的反向 SDE（只依赖 score $\nabla_x\log p_t(x)$）把噪声还原成数据；并给出与之共享边缘分布的确定性 **probability-flow ODE**。

## 数学/技术细节

- 前向 SDE：$\mathrm{d}x = f(x,t)\,\mathrm{d}t + g(t)\,\mathrm{d}w$
- 反向 SDE：$\mathrm{d}x = [f(x,t) - g(t)^2\nabla_x\log p_t(x)]\,\mathrm{d}t + g(t)\,\mathrm{d}\bar w$
- [[wiki/methods/ddpm]] 是 VP-SDE 的离散化，[[wiki/methods/ncsn]] 是 VE-SDE 的离散化——二者在此框架下被统一
- probability-flow ODE 给了确定性采样与精确似然计算的入口——[[wiki/methods/ddim|DDIM]] 的确定性采样（$\sigma=0$）正是它的一个离散化，DDIM 论文给出的 ODE $\mathrm d\bar x=\varepsilon_\theta(\cdot)\,\mathrm d\sigma$ 与此同源

## 与其他概念的关系

- 严格化了 [[wiki/concepts/diffusion-process]] 中「small-$\beta$ ⇒ 反向高斯」的离散近似——回应 DDPM 笔记里 🔴 的 SDE 理论疑虑
- 统一了 [[wiki/concepts/score-matching]] 的离散多尺度版本与连续版本
- 与 [[wiki/concepts/langevin-dynamics]]：反向 SDE / ODE 是 annealed Langevin 的连续时间对应物

## 在 text-guided editing 中的作用

- 提供「editing = 在反向 SDE/ODE 轨迹上注入条件」的连续视角，是理解 inversion 与确定性编辑（如 DDIM inversion）的理论底座

## 待补

- [ ] ingest Yang Song et al. 2021 原文：VP/VE/sub-VP 三类 SDE、predictor-corrector 采样、精确似然
- [ ] 与 probability-flow ODE → DDIM / Flow Matching 的脉络梳理

## 出处与引用

- Yang Song et al. 2021（Score SDE 原文，待 ingest）
- [[wiki/sources/songDenoisingDiffusionImplicit2022]] —— DDIM 的 ODE 视角是 probability-flow ODE 的离散化，二者脉络已对接（精确对应仍待 ingest Yang Song 2021）
- 暂引自 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] 的 open questions 交叉引用
