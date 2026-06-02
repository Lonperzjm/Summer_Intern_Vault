---
type: entity
title: FLUX（具名模型族）
aliases: [FLUX, FLUX.1, FLUX1.1, "FLUX.1 [dev]", "FLUX.1 [pro]"]
tags: [named-model, flow-matching, rectified-flow, dit, latent]
status: draft
created: 2026-06-02
updated: 2026-06-02
sources: ["[[wiki/sources/labsFLUX1KontextFlow2025]]"]
kind: model
---

# FLUX（具名模型族）

## 简介

[[wiki/entities/black-forest-labs|Black Forest Labs]] 的 [[wiki/methods/rectified-flow|rectified flow]] 潜空间 DiT 模型族（SD3 之后的工业主力）：16-channel 潜空间、double+single stream DiT、3D RoPE。是 [[wiki/overview]] 「flow-matching-based 派系」的工业落点之一。

## 谱系 / 变体

- 基础：FLUX.1（`[dev]` 开源权重 / `[pro]` 闭源 / `[schnell]` 蒸馏快采样）；FLUX1.1 [pro]
- 编辑：[[wiki/methods/flux-kontext|FLUX.1 Kontext]]（in-context 生成 + 编辑，[[wiki/sources/labsFLUX1KontextFlow2025|2025]]）

## 关系网

- 机构：[[wiki/entities/black-forest-labs]]
- 谱系：[[wiki/methods/ldm|LDM]] → [[wiki/entities/stable-diffusion|Stable Diffusion]] → SD3 → FLUX
- 训练目标：[[wiki/methods/rectified-flow]] / [[wiki/concepts/flow-matching]]

## 备注

- stub：FLUX.1 / SD3 原文（基础模型）仍待 ingest
