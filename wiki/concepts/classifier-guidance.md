---
type: concept
title: Classifier Guidance（分类器引导）
aliases: [classifier guidance, 分类器引导, guided diffusion, "Dhariwal & Nichol 2021"]
tags: [diffusion, guidance, conditioning]
status: active
created: 2026-05-20
updated: 2026-05-20
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
---

# Classifier Guidance（分类器引导）

> 概念页。数学骨架来自 [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] 的条件生成；"放大引导强度 + 噪声鲁棒分类器"的工程化由 Dhariwal & Nichol 2021（*Diffusion Models Beat GANs*）给出（原文待 ingest）。

## 一句话定义

用一个**独立训练的分类器** $p_\phi(y\mid x_t)$（在带噪图像上训练）提供条件梯度，在采样时把它加到无条件 score 上，从而把无条件扩散模型"掰"向目标类别/条件 $y$——无需重训生成模型本身。

## 数学/技术细节

### 贝叶斯分解（来自 Score SDE）

条件 score 可拆为无条件 score 加引导项：
$$\nabla_x\log p_t(x\mid y) = \underbrace{\nabla_x\log p_t(x)}_{\text{score network}} + \underbrace{\nabla_x\log p_t(y\mid x)}_{\text{分类器梯度}}.$$
代入 [[wiki/concepts/score-sde|反向 SDE]] 即得条件反向 SDE（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]] 用它做 class-conditional 生成、inpainting、colorization）：
$$\mathrm dx=\big\{f-g^2[\nabla_x\log p_t(x)+\nabla_x\log p_t(y\mid x)]\big\}\mathrm dt+g\,\mathrm d\bar w.$$

### 引导强度 $s$（Dhariwal & Nichol 2021）

实践中给引导项乘一个 $s>1$ 放大条件性，牺牲多样性换保真：
$$\nabla_x\log p_t(x\mid y) \approx \nabla_x\log p_t(x) + s\,\nabla_x\log p_t(y\mid x).$$
在 [[wiki/concepts/epsilon-parameterization|ε 参数化]] 下等价于
$$\hat\varepsilon = \varepsilon_\theta(x_t,t) - s\sqrt{1-\bar\alpha_t}\,\nabla_{x_t}\log p_\phi(y\mid x_t).$$

## 与 Classifier-Free Guidance 的区别

| | classifier guidance | [[wiki/concepts/classifier-free-guidance\|CFG]] |
|---|---|---|
| 额外模型 | 需单独训练**噪声鲁棒分类器** $p_\phi(y\mid x_t)$ | 无，条件内化进同一网络 |
| 训练成本 | 多一个分类器 + 在带噪数据上训 | 仅条件 dropout，几乎为零 |
| 采样 | 反传过分类器求梯度 | 一次前向算条件/无条件两路 |
| 引导信号 | 任意可微信号（分类器 / CLIP / 分割图）皆可 | 限于训练时见过的条件 $c$ |

CFG 可视作"把引导项内化、免去外部分类器"的演化版；但 classifier guidance 的**外挂任意可微信号**特性，在编辑里反而是优点。

## 在 text-guided editing 中的作用

- 是 overview「编辑层四旋钮」中的 **guidance** 旋钮之一
- 把分类器换成 CLIP/分割/检测等任意可微评分器 → 各种"X-guidance"编辑方法的通用模板（引导项 $\nabla_x\log p_t(y\mid x)$ 即"领域知识注入通道"）

## 待补

- [ ] ingest Dhariwal & Nichol 2021 原文：噪声鲁棒分类器训练、$s$ 与 FID/IS 的 trade-off、ADM 架构
- [ ] 与 CFG 的定量对比（何时各自更优）

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（条件反向 SDE 的贝叶斯分解、逆问题求解）
- Dhariwal & Nichol 2021（classifier guidance + 引导强度，原文待 ingest）
- 演化版：[[wiki/concepts/classifier-free-guidance]]
- 推广版：[[wiki/concepts/energy-guidance]]（把"分类器梯度"推广为"任意能量 + product of experts"，代表 [[wiki/methods/egsde|EGSDE]]）
- 免训练版：[[wiki/concepts/training-free-guidance]]（$\hat x_0$ 点估计 + 现成模型，代表 [[wiki/methods/freedom|FreeDoM]]）；母页 [[wiki/concepts/conditional-diffusion]]
