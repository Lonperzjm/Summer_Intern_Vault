---
type: entity
title: Stable Diffusion
aliases: [SD, Stable Diffusion 1.x, Stable Diffusion 2.x, SD1.5, SDv1.5, SDXL, "stable-diffusion"]
tags: [model, named-model, diffusion, latent-diffusion, text-to-image, foundational]
status: stable
created: 2026-05-28
updated: 2026-05-28
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]", "[[wiki/sources/zhangAddingConditionalControl2023]]"]
kind: model
---

# Stable Diffusion

## 简介

**具名 T2I 模型系列**，基于 [[wiki/methods/ldm|LDM-KL-8]] 架构、用 LAION 子集训练，由 Stability AI（与 [[wiki/entities/compvis|CompVis @ LMU]]、Runway、LAION 合作）2022 起陆续公开发布权重。SD 是绝大多数 text-guided editing 论文的事实底座——它把 LDM 这套"方法"具体化为一组**开源的、可下载的、可二次 fine-tune 的预训练权重**，是 thesis 与所有下游编辑方法的共同起点。

> **方法 vs 模型的区分**：[[wiki/methods/ldm|LDM]] 是配方（autoencoder + ε-pred U-Net + cross-attention），Stable Diffusion 是该配方的**具体落地权重**——本页只描述模型实例化与版本谱系，方法层面见 [[wiki/methods/ldm]]。

## 版本谱系

| 版本 | 发布 | 关键配置 | 重要差异 |
|---|---|---|---|
| **SD 1.0–1.4** | 2022.08 | LDM-KL-8 (f=8)、U-Net、CLIP ViT-L/14 文本编码器、LAION-2B-en aesthetic 子集 | 第一版公开权重 |
| **SD 1.5** | 2022.10（Runway 发布） | 同上 + 额外 595k 步精调 | **[[wiki/methods/controlnet\|ControlNet]] 原文使用的底座**；社区使用最广 |
| **SD 2.0 / 2.1** | 2022.11 / 2022.12 | OpenCLIP ViT-H/14 文本编码器、768² 分辨率版本、新过滤的 LAION-5B 子集 | 文本编码器换 OpenCLIP；过滤更严的训练集；但社区编辑生态多停留在 1.5 |
| **SDXL** | 2023.07 | 双文本编码器（CLIP ViT-L + OpenCLIP ViT-bigG）、更大 U-Net、refiner 模型、原生 1024² | 显著质量提升；仍属 LDM 范式 |
| **SD3** | 2024.x（Esser et al. 2024，待 ingest） | **训练目标改为 [[wiki/methods/rectified-flow\|Rectified Flow]]**；backbone 由 U-Net 改 MM-[[wiki/sources/peeblesScalableDiffusionModels2023\|DiT]] | **离开 ε-pred / U-Net**，进入 RF + DiT 时代 |

> 关键 ingest 缺口：**SD3** 与 **FLUX**（Black Forest Labs）—— overview「主要派系→flow-matching-based」与「可变性光谱→训练目标」两条线的当前最佳落点。

## 关键贡献 / 地位

- **学界事实底座**：text-guided editing 90%+ 的方法（Prompt-to-Prompt、Null-text inversion、InstructPix2Pix、ControlNet、IP-Adapter、Plug-and-Play、MasaCtrl、StyleAligned、…）均默认 SD1.5 / SDXL 之上
- **资源民主化**：开源权重 + 单 GPU 可推断，是学界做生成 / 编辑实验的可行性前提；本身就是 thesis "高校实验室资源约束下的可行路径"这条 implication 能成立的客观条件
- **方法学落地点**：把 [[wiki/methods/ldm|LDM]] 的"两阶段管线 + cross-attention 注入"具体化为社区可调用的工具栈（diffusers、ComfyUI、Automatic1111 等）

## 关系网

- 方法基础：[[wiki/methods/ldm|LDM]]（架构与训练范式）
- 关键组件：[[wiki/concepts/perceptual-compression|perceptual compression]] 的 autoencoder（KL-reg, $f=8$）、[[wiki/concepts/cross-attention|cross-attention]] 注入文本、[[wiki/concepts/epsilon-parameterization|ε-prediction]] 训练目标、[[wiki/concepts/classifier-free-guidance|CFG]] 推断标配
- 主要 SD 系作者：[[wiki/entities/robin-rombach]]、Patrick Esser、Andreas Blattmann（CompVis → Stability AI → Black Forest Labs）
- 实验室源头：[[wiki/entities/compvis]] @ [[wiki/entities/lmu-munich]]；下游产品由 Stability AI 维护
- 直接附着 / 利用 SD 的方法（已 ingest）：[[wiki/methods/controlnet|ControlNet]]（附着于 SD 1.5）
- 直接附着 SD 的方法（待 ingest）：Prompt-to-Prompt、Null-text inversion、InstructPix2Pix、T2I-Adapter、IP-Adapter、LoRA、Plug-and-Play、MasaCtrl、StyleAligned、…

## 备注

- 本页是**具名模型**条目（CLAUDE.md §2 entities 定义）；版本谱系会随 SD3 / FLUX ingest 持续更新
- 与 [[wiki/methods/ldm]] **不重复**：方法属性在那边，模型实例属性（权重版本、训练数据、文本编码器选型）在本页
