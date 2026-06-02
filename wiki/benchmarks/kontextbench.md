---
type: benchmark
title: KontextBench
aliases: [KontextBench]
tags: [benchmark, image-editing, in-context]
status: active
created: 2026-06-02
updated: 2026-06-02
sources: ["[[wiki/sources/labsFLUX1KontextFlow2025]]"]
---

# KontextBench

> 数据集 / 评测页。源：[[wiki/sources/labsFLUX1KontextFlow2025|FLUX.1 Kontext]]。vault 首个 **in-context 图像编辑** 评测。

## 一句话

众包真实世界 in-context 任务 benchmark，**1026 image-prompt pairs**，覆盖五类任务，主要用**人评（pairwise / ELO 风格）** + 量化角色相似度。

## 任务构成（五类）

| 任务 | 样本数 |
|---|---|
| local instruction editing | 416 |
| global instruction editing | 262 |
| character reference (CREF) | 193 |
| text editing | 92 |
| style reference (SREF) | 63 |

## 评测方式

- **人评**：跨六个 in-context 任务的偏好对比（Fig 8）。
- **角色保持量化**：用 **AuraFace** 抽编辑前后人脸 embedding，算 cosine 相似度（Fig 8f；多轮 drift 见 Fig 12）。
- 配套 T2I 评测（Internal-T2I-Bench）把质量拆成 prompt following / aesthetic / realism / typography / speed 五维，抵消单一偏好导致的 "bakeyness"（过饱和/中心化/同质 AI 美感）。

## 意义 / 局限

- 覆盖了既有编辑 benchmark 较少涉及的 **character/style reference** 与 **多轮一致性**。
- 以人评为主 → 复现性依赖标注；量化主要限于人脸（AuraFace）。
- 与 I2EBench / EditEval / TEdBench 的覆盖差异待对照（见源页 open questions）。

## 关系

- 源 / 提出模型：[[wiki/sources/labsFLUX1KontextFlow2025]]、[[wiki/methods/flux-kontext]]
- 评测维度参考：[[wiki/benchmarks/metrics]]
