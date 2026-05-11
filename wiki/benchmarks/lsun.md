---
type: benchmark
title: LSUN
aliases: [LSUN-Bedroom, LSUN-Church, LSUN-Cat]
tags: [benchmark, image-generation]
status: draft
created: 2026-05-10
updated: 2026-05-10
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# LSUN

## 简介

Large-scale Scene Understanding Dataset（Yu et al. 2015）。常用 256×256 子集（Bedroom、Church Outdoor、Cat 等）作为高分辨率无条件图像生成的 benchmark。

## 常用指标

- **FID** 主导；偶用 Precision / Recall。

## 标志性结果（diffusion 线索）

- DDPM 在 256×256 LSUN 上达到与 ProgressiveGAN 相当的样本质量（[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §4）

## 出处与引用

- Yu et al. 2015 (LSUN dataset)
- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]
