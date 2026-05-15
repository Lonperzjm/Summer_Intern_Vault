---
type: benchmark
title: CIFAR-10
aliases: [CIFAR10]
tags: [benchmark, image-generation]
status: draft
created: 2026-05-10
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]"]
---

# CIFAR-10

## 简介

60k 张 32×32 彩色图像，10 类，每类 6k；训练 50k / 测试 10k。低分辨率图像生成的事实标准 benchmark。

## 常用指标

- **IS (Inception Score)**：越高越好
- **FID (Fréchet Inception Distance)**：越低越好
- NLL (bits/dim)：似然评估

## 标志性结果（diffusion 线索）

| 方法 | IS ↑ | FID ↓ | 来源 |
|---|---|---|---|
| DDPM | **9.46** | **3.17** | [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] |
| DDIM（$\eta=0$, S=10） | — | 13.36 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |
| DDIM（$\eta=0$, S=50） | — | 4.67 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |
| DDIM（$\eta=0$, S=100） | — | 4.16 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |

> DDIM 的意义不在刷新 SOTA FID，而在 **step–quality trade-off**：用同一个 DDPM 训好的网络，10 步即可出图、50–100 步逼近千步质量。完整分步数表见 [[wiki/sources/songDenoisingDiffusionImplicit2022|原文]] Table 1。
>
> 该表会随后续 ingest 持续追加（IDDPM、ADM、Score SDE、EDM 等）。

## 出处与引用

- Krizhevsky 2009 (CIFAR dataset)
- 在 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §4 中作为主 benchmark
- [[wiki/sources/songDenoisingDiffusionImplicit2022]] §4 中用作分步数采样质量的主 benchmark
