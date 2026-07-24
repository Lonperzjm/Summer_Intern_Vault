---
type: benchmark
title: ImageNet (generation)
aliases: [ImageNet, ImageNet-32, ImageNet-64, ImageNet-128]
tags: [benchmark, image-generation]
status: stable
created: 2026-05-24
updated: 2026-07-24
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# ImageNet (generation)

## 简介

大规模自然图像数据集（Deng et al. 2009），生成研究中常按分辨率切成 ImageNet-32 / 64 / 128 / 256 使用，是比 [[wiki/benchmarks/cifar10|CIFAR-10]] 更能反映**可扩展性**的 benchmark。

## 常用指标

- **FID ↓**、**IS ↑**、**NLL (bits/dim) ↓**；采样效率常用 **NFE**（adaptive ODE solver 平均函数求值，越低越省）。
- 指标定义、方向与缺陷见 [[wiki/benchmarks/metrics]]。

## 标志性结果（Flow Matching 线）

**同架构消融，unconditional（[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]，NLL / FID / NFE）**

| 方法 | ImageNet-32 | ImageNet-64 |
|---|---|---|
| [[wiki/methods/ddpm\|DDPM]] | 3.54 / 6.99 / 262 | 3.32 / 17.36 / 264 |
| Score Matching | 3.56 / 5.68 / 178 | 3.40 / 19.74 / 441 |
| ScoreFlow | 3.55 / 14.14 / 195 | 3.36 / 24.95 / 601 |
| FM w/ Diffusion | 3.54 / 6.37 / 183 | 3.33 / 16.88 / 187 |
| **FM w/ [[wiki/concepts/optimal-transport-path\|OT]]** | **3.53 / 5.02 / 122** | **3.31 / 14.45 / 138** |

**ImageNet-128×128（unconditional FID ↓）**

| 方法 | FID ↓ |
|---|---|
| Uncond. BigGAN | 25.3 |
| PGMGAN | 21.7 |
| **FM w/ OT** | **20.9**（NLL 2.90） |

> FM-OT 是当时 ImageNet-128 无条件 SOTA（仅次于用 self-sup 条件的 IC-GAN，不同设定）。

**条件超分辨率 64→256（ImageNet val，[[wiki/sources/lipmanFlowMatchingGenerative2023|FM]] vs SR3）**

| 方法 | FID ↓ | IS ↑ | PSNR | SSIM |
|---|---|---|---|---|
| SR3 | 5.2 | 180.1 | 26.4 | 0.762 |
| **FM w/ OT** | **3.4** | **200.8** | 24.7 | 0.747 |

> FM-OT 在 FID / IS 上明显优于 SR3，PSNR/SSIM 相当——作者引 Saharia et al. 2022 论点：FID/IS 更能反映生成质量。

## 出处与引用

- Deng et al. 2009（ImageNet dataset）
- [[wiki/sources/lipmanFlowMatchingGenerative2023]] §6（主 benchmark：密度建模、采样效率、条件超分）
- 该表会随后续 ingest 追加（ADM / Score SDE / VDM / SD3 等）
