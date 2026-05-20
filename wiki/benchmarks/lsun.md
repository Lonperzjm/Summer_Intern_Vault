---
type: benchmark
title: LSUN
aliases: [LSUN-Bedroom, LSUN-Church, LSUN-Cat]
tags: [benchmark, image-generation]
status: draft
created: 2026-05-10
updated: 2026-05-20
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]"]
---

# LSUN

## 简介

Large-scale Scene Understanding Dataset（Yu et al. 2015）。常用 256×256 子集（Bedroom、Church Outdoor、Cat 等）作为高分辨率无条件图像生成的 benchmark。

## 常用指标

- **FID** 主导；偶用 Precision / Recall。定义见 [[wiki/benchmarks/metrics]]。

## 标志性结果（diffusion 线索）

- DDPM 在 256×256 LSUN 上达到与 ProgressiveGAN 相当的样本质量（[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §4）
- DDIM 在 LSUN Bedroom / Church 上用 50–100 步采样即可逼近 1000 步 DDPM 的 FID，并展示固定 $x_T$ 下的 consistency 与 latent 插值（[[wiki/sources/songDenoisingDiffusionImplicit2022]]，分步数 FID 见原文 Table 2）

## 出处与引用

- Yu et al. 2015 (LSUN dataset)
- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]
- [[wiki/sources/songDenoisingDiffusionImplicit2022]]
