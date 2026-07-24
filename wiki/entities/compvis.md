---
type: entity
title: CompVis Group
aliases: [CompVis, CompVis Lab, Computer Vision & Learning Group]
tags: [lab]
status: stable
created: 2026-05-27
updated: 2026-07-24
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"]
kind: lab
---

# CompVis Group

## 简介

[[wiki/entities/bjorn-ommer]] 领导的 Computer Vision & Learning Group。早期在 Heidelberg University，~2021 后迁至 [[wiki/entities/lmu-munich]]。是 2020–2022 间 **latent generative modeling** 的关键策源地——从 VQGAN 到 Latent Diffusion Models 到 Stable Diffusion 的核心工作链均出自此组。

## 关键贡献 / 关键工作

- **VQGAN / "Taming Transformers for High-Resolution Image Synthesis"**（Esser, Rombach & Ommer 2021，仍待 ingest）—— autoencoder + adversarial 训练，建立 perceptual compression 的工程范式；是 LDM 的直接前身
- **Latent Diffusion Models**（[[wiki/sources/rombachHighResolutionImageSynthesis2022]]）—— 本组里程碑
- 多个 CVPR / ICCV / NeurIPS 在生成模型、representation learning 方向的工作

## 关系网

- PI：[[wiki/entities/bjorn-ommer]]
- 关键成员（LDM/SD 系）：[[wiki/entities/robin-rombach]]、Patrick Esser、Andreas Blattmann、Dominik Lorenz
- 后续轨迹：核心成员陆续加入 Stability AI（SD1/2 时期）→ Black Forest Labs（FLUX）/ 留在学界

## 备注

- 与 [[wiki/methods/ldm|LDM]] 与 Stable Diffusion 强绑定；几乎所有 text-guided editing 论文都建在该组工作的底座上
