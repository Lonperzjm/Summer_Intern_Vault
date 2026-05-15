---
type: concept
title: Score Matching
aliases: [denoising score matching, DSM, score-based]
tags: [diffusion, score-based]
status: active
created: 2026-05-10
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Score Matching

## 一句话定义

直接学习数据（或扰动数据）对数密度的梯度 $\nabla_x \log p(x)$（"score"），从而绕开归一化常数；denoising score matching 进一步把它转化为对噪声的最小二乘回归。

## 数学/技术细节

### Score 的定义

$$s(x) := \nabla_x \log p(x)$$

学习目标若直接最小化 $\mathbb{E}_p \|s_\theta(x)-s(x)\|^2$ 不可行（不知 $s$）。Hyvärinen 2005 给出 implicit score matching；Vincent 2011 证明它等价于 denoising score matching：

$$\mathcal{L}_{DSM}(\theta) = \mathbb{E}_{x,\tilde x}\,\|s_\theta(\tilde x) - \nabla_{\tilde x}\log q_\sigma(\tilde x\mid x)\|^2$$

其中 $q_\sigma(\tilde x\mid x)=\mathcal{N}(\tilde x;x,\sigma^2 I)$，$\nabla_{\tilde x}\log q_\sigma = -(\tilde x-x)/\sigma^2$。

### 与 ε-prediction 的等价

在 diffusion 的 forward 下，$x_t \mid x_0 \sim \mathcal{N}(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I)$，所以

$$\nabla_{x_t}\log q(x_t\mid x_0) = -\frac{x_t - \sqrt{\bar\alpha_t}x_0}{1-\bar\alpha_t} = -\frac{\varepsilon}{\sqrt{1-\bar\alpha_t}}$$

故 [[wiki/concepts/epsilon-parameterization]] 中预测 $\varepsilon$ 与预测 score 在尺度因子下等价：

$$s_\theta(x_t,t) = -\frac{\varepsilon_\theta(x_t,t)}{\sqrt{1-\bar\alpha_t}}$$

DDPM 的 $L_\mathrm{simple}$ 即一种带特定权重的 multi-noise-level DSM。

## 与其他概念的关系

- 采样：score 给出 [[wiki/concepts/langevin-dynamics]] 步进的方向
- 与 [[wiki/concepts/variational-bound-elbo]] 在 diffusion 框架下殊途同归
- 在 SDE 视角下统一为 score-based generative modeling（[[wiki/concepts/score-sde|Yang Song et al. 2021]]）

## 在 text-guided editing 中的作用

- 提供"editing = 在 score 场中沿条件方向行走"的几何直觉；[[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance]] 可视作把 score 替换为 $s_\theta(x_t,t,c) + w(s_\theta(x_t,t,c)-s_\theta(x_t,t,\emptyset))$

## 出处与引用

- Vincent 2011（DSM）；Hyvärinen 2005（ISM）；[[wiki/methods/ncsn|Song & Ermon 2019]]（NCSN，多噪声尺度 DSM）
- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §3.2 与 §4.2 建立桥
