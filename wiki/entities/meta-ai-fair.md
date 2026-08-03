---
type: entity
title: Meta AI (FAIR)
aliases: [Meta AI, FAIR, Facebook AI Research, Meta]
tags: [institution]
status: stable
created: 2026-05-24
updated: 2026-07-31
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]", "[[wiki/sources/shaulBespokeSolversGenerative2023]]", "[[wiki/sources/shaulBespokeNonStationarySolvers2024]]"]
kind: org
---

# Meta AI (FAIR)

## 简介

Meta（原 Facebook）的 AI 研究机构 Fundamental AI Research。在生成式建模上，[[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 出自此处——把连续归一化流（[[wiki/concepts/continuous-normalizing-flow|CNF]]）的研究线接进 diffusion 主线，催生后续 rectified-flow 一系。

## 关键贡献 / 关键工作

- [[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching for Generative Modeling]]（2023）—— 提出 [[wiki/concepts/flow-matching|FM]] / [[wiki/concepts/optimal-transport-path|OT 路径]]
- [[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solvers]]（2023）/ [[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS]]（2024）—— FM/diffusion 专用 solver 优化（Lipman 组）

## 关系网

- 人物：[[wiki/entities/yaron-lipman]]、[[wiki/entities/ricky-chen]]、Neta Shaul、Heli Ben-Hamu、Maximilian Nickel、Matt Le

## 备注

- Bespoke / BNS solver 线是 Lipman 组在 FM 之后的重要延伸：从"提出框架"到"优化框架的推理效率"
