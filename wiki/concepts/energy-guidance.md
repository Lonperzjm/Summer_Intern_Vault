---
type: concept
title: Energy Guidance（能量引导）
aliases: [energy guidance, 能量引导, energy-guided sampling, product of experts guidance]
tags: [diffusion, guidance, conditioning, energy-guidance]
status: active
created: 2026-06-21
updated: 2026-06-21
sources: ["[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]"]
---

# Energy Guidance（能量引导）

> 概念页。把 [[wiki/concepts/classifier-guidance|classifier guidance]] 从"单个分类器对数似然梯度"推广为"任意能量函数 $\mathcal E$ 的梯度"，在采样期注入条件、**不重训生成器**。代表方法 [[wiki/methods/egsde|EGSDE]]（[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022|Zhao et al. 2022]]）。

## 一句话定义

在反向扩散/SDE 采样时，用一个预定义/预训练的**能量函数** $\mathcal E(x_t,c,t)$ 的梯度修正 score：
$$s_{\text{guided}}(x_t,t,c)=s(x_t,t)-\nabla_{x_t}\mathcal E(x_t,c,t),$$
等价于从 $\propto p_t(x_t)\,e^{-\mathcal E(x_t,c,t)}$ 采样。能量越低的样本越被偏好，从而把无条件模型"掰"向条件 $c$。

## 与 classifier guidance 的关系

- **classifier guidance** 是特例：取 $\mathcal E=-\log p_\phi(y\mid x_t)$，则 $-\nabla\mathcal E=\nabla\log p_\phi(y\mid x_t)$ 正是分类器梯度。
- **energy guidance** 放宽到**任意可微能量**：余弦相似度、L2、CLIP score、reward、判别器 logits……都能当 energy。这把"领域知识注入通道"从"一个分类器"扩成"一族能量"。
- 多个能量相加 ⇔ **product of experts**：$e^{-\sum_k\lambda_k\mathcal E_k}=\prod_k e^{-\lambda_k\mathcal E_k}$，每个专家独立贡献一个性质（如 EGSDE 的 realism / faithfulness 三专家）。

## 设计轴：评分在 noisy 还是 clean 空间

能量要对 noisy 的 $x_t$ 求梯度，但判别信号往往定义在 clean 图上。两条路线：

| | noisy-aligned | clean-estimate |
|---|---|---|
| 做法 | 把参照也噪化到同 $t$，noisy-to-noisy 比 | 先估 $\hat x_0(x_t,t)$（Tweedie/一步），在 clean 空间评分 |
| 代表 | [[wiki/methods/egsde\|EGSDE]] 的 $\mathcal E(y_t,x_t,t)$ | [[wiki/methods/freedom\|FreeDoM]] / Universal Guidance / DPS / LGD |
| 评分器 | 须 **noise-aware 重训** | 可**复用现成 clean 模型**（零训练） |
| 弱点 | 每 task 重训、信号受限 | $\hat x_0$ 高噪声下糊 → 梯度不稳 |

> 这条轴是 [[research/ideas]] energy-guidance 候选的核心：能否在**不重训**前提下把高噪声 $\hat x_0$ 评分做准。[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]] 的 $\hat x_0=x_t-t\,v_\theta$ 因轨迹近直线更干净，是该轴上一个有利但未验证的切入点。

## 在条件生成 / 编辑中的作用

- 与 [[wiki/methods/sdedit|SDEdit]]（只改起点）正交：energy guidance 在**每个 $t$** 持续注入条件梯度。
- 与 [[wiki/methods/controlnet|ControlNet]]（改网络再训练）/ [[wiki/concepts/in-context-conditioning|in-context]]（拼 token）相比，energy guidance 走**冻结生成器 + 采样期梯度**这一路——工程上最轻、可即插任意可微信号。

## 待补

- [ ] ingest training-free guidance 统一框架 [TFG (2403.12404)](https://arxiv.org/abs/2403.12404) 与 flow 版 [TFG-Flow (2501.14216)](https://arxiv.org/pdf/2501.14216)
- [ ] energy guidance 的合法性条件（何时 $\propto p_t e^{-\mathcal E}$ 仍是良定的反向 SDE）

## 出处与引用

- [[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]（EGSDE：能量引导 + PoE 的奠基样本）
- [[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]（FreeDoM：clean-estimate 免训练能量引导）
- 原型：[[wiki/concepts/classifier-guidance]]；演化：[[wiki/concepts/classifier-free-guidance]]；母页：[[wiki/concepts/conditional-diffusion]]；免训练子族：[[wiki/concepts/training-free-guidance]]
