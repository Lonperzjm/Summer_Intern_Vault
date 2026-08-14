---
type: concept
title: Training-Free Guidance（免训练引导）
aliases: [training-free guidance, 免训练引导, plug-and-play guidance, clean-estimate guidance, TFG, "Training-free Diffusion Guidance"]
tags: [diffusion, guidance, conditioning, energy-guidance, training-free]
status: active
created: 2026-06-23
updated: 2026-08-14
sources: ["[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]", "[[wiki/sources/jainDiffusionTreeSampling2025]]"]
---

# Training-Free Guidance（免训练引导）

> 概念页。指**不重训生成器、也不训练 noise-aware 打分器**，仅靠"估 $\hat x_0$ + 现成 clean-image 模型"在采样期注入条件的一族方法。代表 [[wiki/methods/freedom|FreeDoM]]、DPS；统一框架 TFG。

## 一句话定义

判别器/reward 都在**干净图**上训过，但采样时在带噪 $x_t$。免训练引导 = 用 Tweedie 估 $\hat x_0=\hat x_0(x_t,t)$，把现成模型评在 $\hat x_0$ 上，梯度 $\nabla_{x_t}E(\hat x_0,c)$ 当引导——**不为新条件训练任何东西**（详见 [[wiki/concepts/conditional-diffusion]] §3–§4）。

## 与 EGSDE / classifier guidance 的分界

- [[wiki/concepts/classifier-guidance|classifier guidance]] / [[wiki/methods/egsde|EGSDE]]：要**重训 noise-aware 打分器**（吃 $(x_t,t)$）。换个条件就重训——贵。
- training-free：复用**现成 time-independent** 模型（CLIP / ArcFace / 分割 / 检测 …），零额外训练。代价是 $\hat x_0$ 高噪声下糊 → 梯度有偏、不稳。

## 设计空间（TFG 统一表）

[TFG (Ye et al., NeurIPS'24, 2403.12404)](https://arxiv.org/abs/2403.12404) 证明这族方法是**同一个设计空间**的特例，四个旋钮：

| 旋钮 | 含义 |
|---|---|
| **mean guidance** | 把能量评在 $\hat x_0$ 上（所有方法都做） |
| **variance guidance** | 在 $\hat x_0$ 周围加噪/采样平滑梯度 |
| **recurrence / time-travel** | 回跳几步重评引导 |
| **iteration / 步长** | 自适应步长（Polyak）促收敛 |

各方法 = 限制超参的特例：

| 方法 | $\hat x_0$ | 期望怎么处理 | 额外 trick |
|---|---|---|---|
| **DPS** | Tweedie | 点估计 | 反传过网络（反问题） |
| **[[wiki/methods/freedom\|FreeDoM]]** | Tweedie | 点估计 | **time-travel**（仅 semantic stage） |
| **Universal Guidance** | Tweedie | 点估计 | forward+backward + self-recurrence |
| **LGD** | Tweedie | **局部 MC**（$\mathcal N(\hat x_0,r^2I)$ 采样） | variance guidance |
| **MPGD** | latent Tweedie | 点估计 | 流形投影，免反传扩散网 |

> 轴心区别：**FreeDoM/UG/DPS = 确定性点估计**；**LGD = 采样估期望**。要快速吃下这一片，读 TFG 一篇即可。

## 偏差问题与 flow 角度

点估计 $E(\hat x_0)\approx\mathbb E[E(x_0)]$ 有 Jensen 偏差，高噪声下大。修法：训练精确能量（[Contrastive Energy Prediction (Lu 2023)](https://arxiv.org/pdf/2304.12824)）或局部 MC（LGD）。

[[wiki/methods/diffusion-tree-sampling|Diffusion Tree Sampling]] 给出更全局的替代：不用一次 $r(\hat x_0)$ 决策，而是保存 stochastic denoising tree，以多个真实 terminal rewards 做 soft-value backup。其 2D 实验直接显示 Tweedie one-step 在高噪声时 bias/variance 上升，而 tree aggregation 可降低两者；代价是大量串行 rollout、缓存和 branching NFE。
- **flow 角度**：FM 无显式 score，[[wiki/methods/fmps|FMPS]] 用"速度↔score 桥"把这套接到 flow，并给 $\hat x_0=x_t-tv$（gradient，准/贵）与前向反解（free，便宜/糙）两版——**等于把"flow 上 clean-estimate 引导"做齐了**。其它 flow 版：[TFG-Flow](https://arxiv.org/pdf/2501.14216)、FlowChef、OC-Flow。⚠️ 注：$\hat x_0=x_t-tv=\mathbb E[x_0\mid x_t]$ 同为后验均值，"flow 更准"未被证（[[research/ideas]]）。

## 关系

- 母页：[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/energy-guidance]]
- 原型：[[wiki/concepts/classifier-guidance]]；对照：[[wiki/methods/egsde]]（noisy-aligned 反例）
- 代表方法：[[wiki/methods/freedom]]
- value-backup / search 路线：[[wiki/methods/diffusion-tree-sampling|DTS]]
- 出处：[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]
