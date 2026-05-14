---
type: concept
title: Classifier-Free Guidance (CFG)
aliases: [CFG, classifier-free guidance, 无分类器引导]
tags: [diffusion, guidance, conditioning]
status: draft
created: 2026-05-14
updated: 2026-05-14
sources: []
---

# Classifier-Free Guidance (CFG)

> Stub 页。Ho & Salimans, *Classifier-Free Diffusion Guidance*, 2022。等 ingest 原始文献后扩充。

## 一句话定义

不训练单独的分类器，而是**同一个网络同时学条件与无条件 score**（训练时随机丢弃条件 $c$），采样时把两者外推：
$$\tilde\varepsilon_\theta(x_t,t,c) = \varepsilon_\theta(x_t,t,\varnothing) + w\big(\varepsilon_\theta(x_t,t,c) - \varepsilon_\theta(x_t,t,\varnothing)\big)$$
通过引导强度 $w$ 在样本多样性与条件贴合度之间换挡。

## 数学/技术细节

- 等价于把 score 替换为 $s_\theta(x_t,t,c) + w\big(s_\theta(x_t,t,c)-s_\theta(x_t,t,\varnothing)\big)$（见 [[wiki/concepts/score-matching]]）
- 训练成本几乎为零：只需在条件 dropout 下复用 [[wiki/concepts/epsilon-parameterization]] 的 $L_\mathrm{simple}$
- 是现代 text-to-image（Imagen / Stable Diffusion）与 text-guided editing 的事实标准

## 与其他概念的关系

- 作用在 [[wiki/concepts/diffusion-process]] 的 reverse 链上，是 overview「编辑层四旋钮」中的 **guidance** 旋钮
- 前身：classifier guidance（Dhariwal & Nichol 2021）

## 在 text-guided editing 中的作用

- 几乎所有编辑方法默认开启 CFG；引导强度、对正/负 prompt 的差异化加权本身就是一个可调研的编辑杠杆

## 待补

- [ ] ingest Ho & Salimans 2022 原文：条件 dropout 比例、$w$ 与 FID/IS 的 trade-off 曲线
- [ ] classifier guidance（Dhariwal & Nichol 2021）单独成页或并入

## 出处与引用

- Ho & Salimans 2022（CFG 原文，待 ingest）
- 关联人物：[[wiki/entities/jonathan-ho]]
