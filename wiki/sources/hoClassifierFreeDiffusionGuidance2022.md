---
type: source
title: "Classifier-Free Diffusion Guidance"
aliases: [CFG paper, "Ho & Salimans 2022"]
tags: [diffusion, guidance, conditioning]
status: active
created: 2026-08-11
updated: 2026-08-11
raw: "[[raw/literature-notes/hoClassifierFreeDiffusionGuidance2022]]"
authors: "Jonathan Ho, Tim Salimans"
venue: NeurIPS 2021 Workshop (preprint arXiv 2207.12598)
year: 2022
arxiv: "2207.12598"
---

# Classifier-Free Diffusion Guidance

> Ho & Salimans, 2022. 提出 **Classifier-Free Guidance (CFG)**——不训练单独分类器，而是让同一个网络同时学条件与无条件 score，采样时外推二者差值。此后几乎成为所有条件 diffusion / text-to-image 的标准组件。

## Motivation

[[wiki/sources/dhariwalDiffusionModelsBeat2021|Dhariwal & Nichol 2021]] 的 [[wiki/concepts/classifier-guidance|classifier guidance]] 证明了在采样时加入分类器梯度可以显著提升条件保真度，但需要：

1. 额外训练一个噪声鲁棒分类器 $p_\phi(y\mid x_t, t)$；
2. 该分类器必须在每个 noise level 下工作（不能直接用 clean-image 分类器）。

作者的问题：**能不能完全去掉外部分类器，只靠生成模型自身实现 guidance？**

## Method

### 1. 条件 dropout 训练

只训练一个网络 $\varepsilon_\theta(x_t, t, c)$。训练时以概率 $p_\text{uncond}$ 将条件 drop 为空：

$$c \leftarrow \varnothing \quad \text{with probability } p_\text{uncond}.$$

于是同一网络有时学 $\varepsilon_\theta(x_t, t, c)$（条件），有时学 $\varepsilon_\theta(x_t, t, \varnothing)$（无条件）。text-to-image 中 $\varnothing$ 对应空文本 embedding。

### 2. CFG 采样公式

采样时每步做两次 forward，然后外推：

$$\boxed{\tilde\varepsilon = \varepsilon_\theta(x_t, t, \varnothing) + s\big(\varepsilon_\theta(x_t, t, c) - \varepsilon_\theta(x_t, t, \varnothing)\big)}$$

其中 $s = 1 + w$ 为 guidance scale（$s=1$ 即无 guidance；Stable Diffusion 常用 $s=7.5$）。

等价的 score 写法：$\tilde s = (1+w)\,s_c - w\,s_u$。

### 3. 隐式分类器推导

在 exact score 下，由 Bayes：

$$\nabla_{x_t}\log p(c\mid x_t) = \nabla_{x_t}\log p(x_t\mid c) - \nabla_{x_t}\log p(x_t) = s_c - s_u.$$

CFG 因此等价于 classifier guidance 用这个 **implicit classifier** $\propto p(x_t\mid c)/p(x_t)$ 的梯度。

### 4. 非保守场 caveat

上述等式**仅在 exact score 下成立**。实际 learned score $s_\theta(x,c)$ 和 $s_\theta(x,\varnothing)$ 是无约束神经网络输出，其差 $s_\theta(x,c) - s_\theta(x,\varnothing)$ **不一定是保守场**（不一定 $=\nabla f$ for some scalar $f$）。所以实际 CFG 并不严格等价于"沿某个隐式分类器的梯度走"——隐式分类器是 motivation，不是精确等式。这一理论缺口是后续 CFG rescaling、dynamic CFG、guidance distillation 等工作的出发点。

### 5. 采样代价

每个 denoising step 需两次 network evaluation（条件 + 无条件），与 sampler 选择（DDPM / DDIM / ODE）正交但有 2× 计算开销。

## Results

ImageNet 64×64 / 128×128 class-conditional generation，$T=256$ steps：

| 设置 (128×128) | FID↓ | IS↑ |
|---|---|---|
| $w=0$（无 guidance） | 7.27 | 82.45 |
| $w=0.3$（最优 FID） | **2.43** | 158.47 |
| $w=4$（强 guidance） | 21.53 | 421.03 |

关键发现：

- **Guidance scale 不是越大越好**：小量 $w$ 大幅降低 FID；继续增大则 IS 暴涨但 FID 回升、diversity 崩塌。
- **最佳 dropout 比例**：$p_\text{uncond} \in [0.1, 0.2]$ 优于 $0.5$，只需少量 capacity 学 unconditional 即可。
- **CFG 与 classifier guidance 效果相当**：在同等 IS/FID trade-off 上二者接近。

## 关系

### 直接前身

- [[wiki/sources/dhariwalDiffusionModelsBeat2021]]：classifier guidance 原文，CFG 是其"去分类器"演化版
- [[wiki/concepts/classifier-guidance]]：CFG 的"外部梯度"前身

### 概念

- [[wiki/concepts/classifier-free-guidance]]：本文定义的核心概念
- [[wiki/concepts/score-matching]] / [[wiki/concepts/score-sde]]：数学底座（贝叶斯分解）
- [[wiki/concepts/epsilon-parameterization]]：训练目标不变，guidance 只在采样时修改 $\hat\varepsilon$

### 后续发展

- CFG rescaling / dynamic CFG / negative prompt weighting：针对 "非保守场 + 过饱和" 的工程修复
- Guidance distillation：把 2-pass CFG 蒸馏为 1-pass，消除 2× 推理开销
- [[wiki/concepts/training-free-guidance]] / [[wiki/concepts/energy-guidance]]：把"条件梯度"从 implicit 推广回"任意外部 energy"

### 实体

- [[wiki/entities/jonathan-ho]]：第一作者，也是 [[wiki/methods/ddpm|DDPM]] 作者
- Tim Salimans：第二作者，Google Brain

### 下游

- [[wiki/methods/ldm|Stable Diffusion]] / Imagen / DALL·E 2 均以 CFG 为标配
- 编辑方法中 CFG 是四旋钮之一（overview「研究杠杆」）

## 对我的 thesis 的启示

1. **不改变 working thesis 版本号**。CFG 是 overview 推论 2（fidelity↔controllability 旋钮）早已引用的组件，此次 ingest 的价值是给概念页补全原文机制与实证。

2. **非保守场 caveat 与 thesis 方向的潜在联系**：当前 reject-and-skip 主线关注的是 conditional velocity field 平均化造成的 ambiguity；而 CFG 的外推本质是放大 $(v_\text{cond} - v_\text{uncond})$。如果这个差值本身在高 ambiguity 区域不可靠（非保守、方向碎片化），那 CFG 过大时的 off-manifold 行为可能与 velocity 不可信区域有关——是潜在的诊断联系点。

3. **$p_\text{uncond}$ 的极简设计对实验的启示**：只用 10% dropout 就足够生成有效的 guidance direction，说明 unconditional branch 学的是"数据流形的基础结构"——这与 reject-and-skip 需要的"marginal velocity $v_\text{uncond}$"作为 baseline 可信参照有直觉上的呼应。

## 待后续 ingest

- [ ] Imagen（Saharia et al. 2022）—— CFG 在 text-to-image 的首个大规模应用
- [ ] CFG rescaling / dynamic thresholding —— 工程修复 oversaturation
