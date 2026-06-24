---
type: benchmark
title: AFHQ（Animal FaceS-HQ）
aliases: [AFHQ, Animal Faces-HQ, "Cat→Dog", "Wild→Dog"]
tags: [dataset, unpaired-i2i, translation, image-editing]
status: stable
created: 2026-06-21
updated: 2026-06-21
sources: ["[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]"]
---

# AFHQ（Animal Faces-HQ）

> 数据集页。高分辨率动物脸数据集，是 **unpaired image-to-image translation** 的标准 benchmark；由 StarGAN v2（Choi et al. 2020）提出，原文待 ingest。

## 概况

- 三个域：**Cat / Dog / Wild**（野生动物）。每域约 5000 张，512×512 高清动物脸。
- 用途：unpaired I2I 翻译（域间无配对）。常用任务设置：**Cat→Dog**、**Wild→Dog**。
- 配套人脸数据集 **CelebA-HQ** 常一起用于 **Male→Female** 翻译任务。

## 在本 wiki 中的使用

- [[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022|EGSDE]]：在 Cat→Dog（FID 51.04）、Wild→Dog（FID 50.43）、CelebA-HQ Male→Female 三任务上评测，三任务 × 四指标（realism = [[wiki/benchmarks/metrics|FID]]；faithfulness = L2 / PSNR / SSIM 等）。
- 也是 [[wiki/methods/rectified-flow|Rectified Flow]] 等 I2I 实验的常见数据集（cat↔dog）。

## 评测维度（unpaired I2I 特有）

unpaired I2I 没有 ground-truth 配对，故同时量两端：
- **realism**：输出像不像目标域真实图 → [[wiki/benchmarks/metrics|FID]]。
- **faithfulness**：输出是否忠于源图结构 → L2 / PSNR / SSIM。

两者天然冲突，方法通常给一个 realism↔faithfulness 旋钮（EGSDE 是 $\lambda_s,\lambda_i$；[[wiki/methods/sdedit|SDEdit]] 是 $t_0$）。

## 关联

- 出处（本 wiki 首次引入）：[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]
- 指标：[[wiki/benchmarks/metrics]]
- 相关方法：[[wiki/methods/egsde]]、[[wiki/methods/sdedit]]、[[wiki/methods/rectified-flow]]
- 待补：StarGAN v2 原文（AFHQ 提出处）、CelebA-HQ 独立页
