---
type: entity
title: Jascha Sohl-Dickstein
aliases: [Sohl-Dickstein, J. Sohl-Dickstein, Jascha]
tags: [researcher]
status: draft
created: 2026-05-20
updated: 2026-05-20
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
kind: person
---

# Jascha Sohl-Dickstein

## 简介

diffusion 生成模型的**鼻祖**。物理背景出身，2015 年从非平衡热力学借来"逐步加噪 / 逐步去噪"的思想，第一个把它形式化为生成模型——这条路线沉寂数年后由 [[wiki/methods/ddpm|DDPM]] 推上 SOTA，如今成为整个领域的基石。长期在 Google Brain。

## 关键贡献 / 关键工作

- [[wiki/methods/diffusion-2015|Diffusion Probabilistic Models（Sohl-Dickstein et al. 2015）]] —— diffusion 原型，前向/反向双 Markov 链 + "small-$\beta$ ⇒ 反向高斯"引理（被 DDPM 直接引用）
- [[wiki/concepts/score-sde|Score SDE]]（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]，合作者）—— 连续时间统一框架

## 关系网

- Score SDE 合作者：[[wiki/entities/yang-song]]、[[wiki/entities/stefano-ermon]] 等
- 其 2015 原型经 [[wiki/methods/ddpm|DDPM]]（Ho et al.）与 [[wiki/methods/ncsn|NCSN]]（Song & Ermon）发扬光大

## 备注

- stub：待 ingest Sohl-Dickstein et al. 2015 原文后扩充其原始训练目标与推导
