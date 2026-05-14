---
type: method
title: NCSN（Noise Conditional Score Network）
aliases: [NCSN, "Song & Ermon 2019", score-based generative model, annealed-langevin model]
tags: [diffusion, score-based, foundational]
status: draft
created: 2026-05-14
updated: 2026-05-14
sources: []
family: other
---

# NCSN（Noise Conditional Score Network）

> Stub 页。Song & Ermon, *Generative Modeling by Estimating Gradients of the Data Distribution*, NeurIPS 2019。等 ingest 原始文献后扩充。

## 一句话总结

用一个**以噪声尺度为条件的网络** $s_\theta(x,\sigma)$ 估计多尺度扰动数据的 score $\nabla_x\log p_\sigma(x)$，再用 [[wiki/concepts/langevin-dynamics]]（annealed 版本）从大噪声到小噪声逐级采样——是与 [[wiki/methods/ddpm]] 并行、后被统一的另一条 score-based 主线。

## 核心机制

- **多噪声尺度 denoising score matching**：在一族 $\{\sigma_1>\dots>\sigma_L\}$ 上同时训练（见 [[wiki/concepts/score-matching]]）
- **Annealed Langevin sampling**：从大 $\sigma$ 到小 $\sigma$ 依次跑 Langevin，解决低密度区 mixing 慢的问题
- 与 DDPM 的关系：DDPM 的 ε-prediction 与 NCSN 的 score estimation 在尺度因子下等价（[[wiki/concepts/epsilon-parameterization]]）；二者后由 Score SDE 统一（[[wiki/concepts/score-sde]]）

## 待补

- [ ] ingest Song & Ermon 2019 原文：训练细节、$\sigma$ 调度、CIFAR-10 结果
- [ ] NCSNv2（Song & Ermon 2020）的改进点

## 关联

- 概念：[[wiki/concepts/score-matching]]、[[wiki/concepts/langevin-dynamics]]、[[wiki/concepts/score-sde]]
- 并行/对照：[[wiki/methods/ddpm]]
- 上游：[[wiki/methods/diffusion-2015]]
- 出处：待 ingest（暂引自 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] 中的交叉引用）
