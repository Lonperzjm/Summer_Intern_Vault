---
type: benchmark
title: CIFAR-10
aliases: [CIFAR10]
tags: [benchmark, image-generation]
status: stable
created: 2026-05-10
updated: 2026-07-24
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]", "[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# CIFAR-10

## 简介

60k 张 32×32 彩色图像，10 类，每类 6k；训练 50k / 测试 10k。低分辨率图像生成的事实标准 benchmark。高分辨率 / 可扩展性对照见姊妹基准 [[wiki/benchmarks/imagenet]]。

## 常用指标

- **IS (Inception Score)**：越高越好
- **FID (Fréchet Inception Distance)**：越低越好
- NLL (bits/dim)：似然评估

> 各指标的定义、方向与缺陷见 [[wiki/benchmarks/metrics]]。

## 标志性结果（diffusion 线索）

| 方法 | IS ↑ | FID ↓ | 来源 |
|---|---|---|---|
| DDPM | **9.46** | **3.17** | [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] |
| DDIM（$\eta=0$, S=10） | — | 13.36 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |
| DDIM（$\eta=0$, S=50） | — | 4.67 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |
| DDIM（$\eta=0$, S=100） | — | 4.16 | [[wiki/sources/songDenoisingDiffusionImplicit2022]] |
| Score SDE（VP, PC + 架构改进） | **9.89** | **2.20** | [[wiki/sources/songScoreBasedGenerativeModeling2021]]（另 2.99 bits/dim） |

> DDIM 的意义不在刷新 SOTA FID，而在 **step–quality trade-off**：用同一个 DDPM 训好的网络，10 步即可出图、50–100 步逼近千步质量。完整分步数表见 [[wiki/sources/songDenoisingDiffusionImplicit2022|原文]] Table 1。
>
> 该表会随后续 ingest 持续追加（IDDPM、ADM、Score SDE、EDM 等）。

## Flow Matching 同架构消融（NLL / FID / NFE）

> ⚠️ 下表是 [[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]] 的 **apples-to-apples 消融**：所有方法共用同一 ImageNet U-Net（Dhariwal & Nichol 2021）、未为 CIFAR-10 调优，故 FID 整体高于上表 SOTA 数（原文自承）。**应横向比同表内方法、勿与上表直接对比。**

| 方法（同架构） | NLL ↓ (bits/dim) | FID ↓ | NFE ↓ |
|---|---|---|---|
| [[wiki/methods/ddpm\|DDPM]] | 3.12 | 7.48 | 274 |
| Score Matching | 3.16 | 19.94 | 242 |
| ScoreFlow | 3.09 | 20.78 | 428 |
| FM w/ Diffusion | 3.10 | 8.06 | 183 |
| **FM w/ [[wiki/concepts/optimal-transport-path\|OT]]** | **2.99** | **6.35** | **142** |

> 看点：换上 [[wiki/concepts/flow-matching|FM]] 的 OT 路径后，NLL、FID、NFE **三项一致最优**——同架构下既更准、更稳又更省采样。

## 出处与引用

- Krizhevsky 2009 (CIFAR dataset)
- 在 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §4 中作为主 benchmark
- [[wiki/sources/songDenoisingDiffusionImplicit2022]] §4 中用作分步数采样质量的主 benchmark
