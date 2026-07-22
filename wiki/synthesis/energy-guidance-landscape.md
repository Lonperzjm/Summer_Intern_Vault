---
type: synthesis
title: Energy-Guidance Landscape（能量引导选题地图）
aliases: [energy-guidance landscape, 能量引导地图]
tags: [synthesis, energy-guidance, guidance, training-free, thesis]
status: active
created: 2026-06-24
updated: 2026-06-24
sources: ["[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]", "[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]"]
---

# Energy-Guidance Landscape（能量引导选题地图）

> 综述页。沉淀 energy-guidance 候选方向（[[research/ideas]]）的 **prior-art sweep + 师兄三段框架 + 收束结论**。和 [[wiki/synthesis/bridge-sde-editing-landscape]] 同规格。范式：discriminative logits/reward → energy → guidance（[[wiki/concepts/energy-guidance]]）。

## 1. Prior-art sweep（2026-06-24，结论：generic 形式全红）

| 落点 | 状态 | 代表 |
|---|---|---|
| 点估计 $\hat x_0$ + 现成 reward（diffusion） | 🔴 重占 | [[wiki/methods/dps\|DPS]] / [[wiki/methods/freedom\|FreeDoM]] / UGD / LGD / MPGD，被 [TFG (2403.12404)](https://arxiv.org/abs/2403.12404) 统一 |
| energy 搬到 flow（= ①轴） | 🔴 占 | **[[wiki/methods/fmps\|FMPS (2411.07625)]]——FreeDoM 的 FM 版，直占①、连"贵准/便宜糙"两实现都给了**；[Energy-Weighted FM (ICLR'25)](https://arxiv.org/pdf/2503.04975)、Energy-Guided FM、[TFG-Flow](https://arxiv.org/pdf/2501.14216) |
| training-free flow 编辑 | 🔴 红海 | [FlowChef (ICCV'25)](https://github.com/FlowChef/flowchef)、[OC-Flow (ICLR'25)](https://arxiv.org/html/2410.18070v2)、D-Flow |
| energy 编辑（EGSDE 后续） | 🔴 占 | [DragonDiffusion](https://arxiv.org/pdf/2307.02421)、[Contrastive Energy Prediction (Lu'23)](https://arxiv.org/pdf/2304.12824) |

**核心 novelty 点（"高效从 $x_t$ 得 $x_0$ 评分"）也被占**——正是 training-free guidance 子领域的中心命题。
**不 KILL 的唯一理由**：坐师兄 / Long 组 flow 在研线（红海生存三铁律之"导师在研线"）。

## 2. Sliver：师兄三段框架（FreeDoM = baseline）

FreeDoM 流水线 $x_t\xrightarrow{①}\hat x_0\xrightarrow{②}E(\hat x_0,c)\xrightarrow{③}\nabla_{x_t}E$，三段都还 heuristic：

| 段 | FreeDoM 的 heuristic | 原理化方向 | 占用 |
|---|---|---|---|
| ① $\hat x_0$ 估计 | [[wiki/concepts/tweedie-formula\|Tweedie]] 单点 | RF 的 $\hat x_0=x_t-t\,v$ | ☠️ 死：[[wiki/methods/fmps\|FMPS]] 占（gradient/free 两实现）；$\hat x_0$ 同为后验均值、"更准"理论存疑 |
| ② energy 获取 | 单个现成距离 + 简单加权 | **结构化 E**：保/丢拆分但用现成模型拼、不重训 | 🔴 占：TtfDiffusion/DICE（解耦）+ GradOPS（正交） |
| ③ energy→guidance | 手调 $\rho_t$ + 欧氏梯度 + 反传 | 原理化步长 / 流形投影 / 便宜雅可比 | 🔴 占：MPGD/TFG/manifold-CFG + FMPS 的 $g^1$ |

## 3. 结构化 E 是什么（②的核心）

朴素相似度 $E=\lVert\hat x_0-x_0\rVert^2$ → 最小化 = 复制源图，没法翻译/编辑。**结构化** = 拆"保"与"改"、符号相反：
$$E=\underbrace{\lambda_{\text{keep}}\,\mathrm{dist}(F_{\text{keep}}(\hat x_0),F_{\text{keep}}(x_0))}_{\text{拉近：保}}-\underbrace{\lambda_{\text{drop}}\,\mathrm{dist}(F_{\text{drop}}(\hat x_0),F_{\text{drop}}(x_0))}_{\text{推开：改}}$$
[[wiki/methods/egsde|EGSDE]] 就是样本（低通=保 / 域分类器=改），但 $F$ 要训练。例：Cat→Dog（保布局/改物种）、改颜色（保形状/拉向 CLIP）、换风格保身份（保 ArcFace/拉向 Gram）。开口 = **用现成模型拼这个结构、不重训**。

## 4. 收束结论（2026-06-29 更新：已作废）

~~三段原理化方向全指向 flow 底座~~——**塌了**。① 的"flow 更准"理论存疑（$\hat x_0$ 同为后验均值）且被 FMPS 占；③ flow 上也被 FMPS 占。**flow 不是金底座，三段全红。**
**净结论**：energy-guidance 候选公开文献**无 carve**，存活仅靠**导师在研线**——下一步把这张全红地图带回师兄定 execution sliver，不再 armchair（详见 [[research/ideas]]）。

## 5. 评测锚点

unpaired I2I / 编辑常用 [[wiki/benchmarks/afhq|AFHQ]]（Cat→Dog / Wild→Dog）+ CelebA-HQ；指标 realism（[[wiki/benchmarks/metrics|FID]]）+ faithfulness（L2/PSNR/SSIM）。baseline = [[wiki/methods/egsde|EGSDE]] + [[wiki/methods/freedom|FreeDoM]]。

## 关系

- 候选条：[[research/ideas]]（energy-guidance）
- 概念：[[wiki/concepts/energy-guidance]]、[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/tweedie-formula]]
- 方法：[[wiki/methods/egsde]]、[[wiki/methods/freedom]]、[[wiki/methods/dps]]
- 姊妹地图：[[wiki/synthesis/bridge-sde-editing-landscape]]（上一个被 KILL 的方向）
