---
type: concept
title: Reparameterization Trick
aliases: [重参数化, reparam trick]
tags: [foundational, generative-model]
status: stable
created: 2026-05-10
updated: 2026-05-10
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Reparameterization Trick

## 一句话定义

把"从分布采样"改写为"对一个无参分布采样后的确定性变换"，从而让随机性从计算图中独立出来，使梯度可通过期望传递。

## 数学/技术细节

若 $z\sim\mathcal{N}(\mu,\sigma^2)$，写成 $z=\mu+\sigma\,\varepsilon,\ \varepsilon\sim\mathcal{N}(0,1)$。期望对 $\mu,\sigma$ 的梯度即可与采样 $\varepsilon$ 解耦。

### 在 diffusion 中的两处用法

1. **Forward 闭式采样**（关键）：由 [[wiki/concepts/diffusion-process]]，

   $$x_t = \sqrt{\bar\alpha_t}\,x_0 + \sqrt{1-\bar\alpha_t}\,\varepsilon,\quad \varepsilon\sim\mathcal{N}(0,I)$$

   一行采到任意 $t$ 步噪声样本，无需展开 $T$ 步链。这使训练时可以对 $t\sim\mathcal{U}\{1,T\}$ 做 Monte Carlo 估计 ELBO。
2. **Reverse 采样**：$x_{t-1}=\mu_\theta+\sigma_t z$。

## 与其他概念的关系

- 是 [[wiki/concepts/variational-bound-elbo]] 训练能高效进行的前提
- 与 [[wiki/concepts/epsilon-parameterization]] 互为镜像：forward 用 $\varepsilon$ 生成 $x_t$，网络反过来从 $x_t$ 预测 $\varepsilon$

## 出处与引用

- Kingma & Welling 2014（VAE，原始 trick）
- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §3（隐式使用）
