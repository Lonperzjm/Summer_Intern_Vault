---
type: benchmark
title: 生成质量评测指标（IS / FID / bits-per-dim）
aliases: [Inception Score, FID, bits-per-dim, IS, NLL, 评测指标]
tags: [evaluation, metrics]
status: active
created: 2026-05-20
updated: 2026-05-24
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]", "[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
---

# 生成质量评测指标

> 汇总页：把现有 source 反复出现的三个**生成质量**指标讲清。注意它们衡量的是"无条件/类条件生成质量"，**不是 text-guided editing 的编辑质量**——后者要 CLIP-T、编辑保真度等指标（待第一篇编辑论文 ingest 时另立页）。

## Inception Score (IS)

- **衡量**：样本的清晰度（单图分类置信度尖锐）× 多样性（类别边缘分布均匀）。
- **定义**：$\mathrm{IS}=\exp\big(\mathbb E_x\,\mathrm{KL}(p(y\mid x)\,\|\,p(y))\big)$，用 Inception 网络的类别分布。
- **方向**：**越高越好**。
- **缺陷**：只看 Inception 类别空间、不与真实数据分布直接比；对类内多样性不敏感；易被对抗样本刷高。

## Fréchet Inception Distance (FID)

- **衡量**：生成分布与真实分布在 Inception 特征空间的距离。
- **定义**：把两组特征各拟合为高斯，算 Fréchet 距离 $\|\mu_r-\mu_g\|^2+\mathrm{Tr}(\Sigma_r+\Sigma_g-2(\Sigma_r\Sigma_g)^{1/2})$。
- **方向**：**越低越好**。
- **缺陷**：高斯假设粗糙；对样本数敏感；仍依赖 Inception 特征，未必对齐人类感知。

## Bits-per-dim (NLL)

- **衡量**：模型对数据的负对数似然，按维度归一化（$-\log_2 p(x)/D$）。
- **方向**：**越低越好**。
- **与本领域的关系**：扩散模型的似然需经 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]（连续归一化流）才能精确计算——[[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] 正是借此报出 CIFAR-10 **2.99 bits/dim**。

## 我们 source 里的数字

| 模型 | 数据集 | IS↑ | FID↓ | bits/dim↓ |
|---|---|---|---|---|
| [[wiki/sources/hoDenoisingDiffusionProbabilistic2020\|DDPM]] | [[wiki/benchmarks/cifar10\|CIFAR-10]] | 9.46 | 3.17 | ≤3.75 |
| [[wiki/sources/songScoreBasedGenerativeModeling2021\|Score SDE]] | [[wiki/benchmarks/cifar10\|CIFAR-10]] | **9.89** | **2.20** | **2.99** |

## 关联

- 数据集：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/lsun]]、[[wiki/benchmarks/imagenet]]
- 精确似然机制：[[wiki/concepts/probability-flow-ode]]
- **待补（编辑专用指标）**：CLIP-T、DINO 结构相似度、编辑保真度 / 背景保持——overview 子问题"CLIP-based 指标的局限与替代"待第一篇 text-guided editing 论文 ingest 时建页
