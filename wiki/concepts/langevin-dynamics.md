---
type: concept
title: Langevin Dynamics（含 annealed 版本）
aliases: [Langevin sampling, annealed Langevin]
tags: [sampling, score-based]
status: active
created: 2026-05-10
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Langevin Dynamics

## 一句话定义

利用 score $\nabla_x \log p(x)$ 做的随机梯度上升采样过程；在足够小步长 + 足够多步数下收敛到 $p$。

## 数学/技术细节

$$x_{k+1} = x_k + \frac{\eta}{2}\nabla_x \log p(x_k) + \sqrt{\eta}\,z_k,\quad z_k\sim\mathcal{N}(0,I)$$

直接用在数据分布上有低密度区域 mixing 慢的问题。**Annealed Langevin**（[[wiki/methods/ncsn|Song & Ermon 2019]]）改为在一族多噪声尺度 $\{\sigma_i\}$ 上从大到小依次采样，每尺度跑若干步 Langevin，类似 simulated annealing。

DDPM 的 reverse step 与 annealed Langevin 在 ε-prediction 框架下结构同构（差一个尺度因子，见 [[wiki/concepts/score-matching]]）。

## 与其他概念的关系

- 步进方向由 [[wiki/concepts/score-matching]] 给出
- 是 [[wiki/concepts/diffusion-process]] reverse 过程的连续/离散对应物
- DDIM 把它替换为确定性常微分方程采样

## 在 text-guided editing 中的作用

- 编辑时常用"在条件 score 场中走 K 步 Langevin"作为 inversion 的精炼步骤

## 出处与引用

- Welling & Teh 2011（SGLD）；[[wiki/methods/ncsn|Song & Ermon 2019]]（annealed Langevin / NCSN）
- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §4.2
