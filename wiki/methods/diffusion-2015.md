---
type: method
title: Diffusion Probabilistic Models（Sohl-Dickstein 2015，原型）
aliases: [Sohl-Dickstein 2015, "diffusion probabilistic model 原型", nonequilibrium thermodynamics diffusion]
tags: [diffusion, foundational]
status: draft
created: 2026-05-14
updated: 2026-05-14
sources: []
family: other
---

# Diffusion Probabilistic Models（Sohl-Dickstein 2015，原型）

> Stub 页。Sohl-Dickstein et al., *Deep Unsupervised Learning using Nonequilibrium Thermodynamics*, ICML 2015。等 ingest 原始文献后扩充。

## 一句话总结

最早把「**用一条逐步加噪的前向链破坏数据、再学一条反向链还原数据**」形式化为生成模型的工作，灵感来自非平衡热力学；原理优雅但样本质量长期落后 GAN/自回归，直到 [[wiki/methods/ddpm]] (2020) 才被推上 SOTA。

## 核心机制

- 前向 / 反向双 Markov 链的原始定义（[[wiki/concepts/diffusion-process]] 的思想来源）
- 反向转移取高斯形式、其「small-$\beta$ 下前后向同形」的引理被 DDPM 直接引用（也是 DDPM 笔记中 🔴 SDE 理论疑虑的源头）

## 待补

- [ ] ingest Sohl-Dickstein 2015 原文：训练目标的原始形式、与 DDPM 的精确差异
- [ ] 厘清「small-$\beta$ ⇒ 反向高斯」引理的严格条件

## 关联

- 概念：[[wiki/concepts/diffusion-process]]、[[wiki/concepts/variational-bound-elbo]]
- 下游：[[wiki/methods/ddpm]]、[[wiki/methods/ncsn]]
- 出处：待 ingest（暂引自 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] 中的交叉引用）
