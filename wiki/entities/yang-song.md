---
type: entity
title: Yang Song
aliases: [Song, Y. Song, Yang Song, 宋飏]
tags: [researcher]
status: stable
created: 2026-05-20
updated: 2026-07-24
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/mengSDEditGuidedImage2022]]"]
kind: person
---

# Yang Song

## 简介

score-based 生成模型这条主线的**奠基人之一**。博士期间在 [[wiki/entities/stanford]]，导师 [[wiki/entities/stefano-ermon]]；后任职 OpenAI。其工作把"用 score 估计 + Langevin 采样做生成"从想法做成 SOTA，并进一步统一进连续时间 SDE 框架。

## 关键贡献 / 关键工作

- [[wiki/methods/ncsn|NCSN（Song & Ermon 2019）]] —— 多噪声尺度 score matching + annealed Langevin，score-based 生成的开山作
- [[wiki/concepts/score-sde|Score SDE]]（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]，第一作者）—— 用 SDE 统一 [[wiki/methods/ncsn|SMLD]] 与 [[wiki/methods/ddpm|DDPM]]，提出 PC 采样与 probability-flow ODE
- [[wiki/methods/sdedit|SDEdit]]（[[wiki/sources/mengSDEditGuidedImage2022]]，合作者）—— noising-based 编辑奠基
- Consistency Models（Song et al. 2023）—— few-step 生成（后续可 ingest）

## 关系网

- 导师：[[wiki/entities/stefano-ermon]]
- **注意区分**：[[wiki/concepts/score-sde|Score SDE]] 的 "Song" 是 **Yang Song**；[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 的 "Song" 是 [[wiki/entities/jiaming-song|Jiaming Song]]——二者均出自 Ermon 组、同为 ICLR 2021，但不是同一人

## 备注

- stub：将在后续 ingest（NCSN、Consistency Models 等）中扩充
