---
type: concept
title: Variational Bound (ELBO) for Diffusion
aliases: [ELBO, 变分下界, L_vlb]
tags: [diffusion, training-objective]
status: active
created: 2026-05-10
updated: 2026-05-10
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Variational Bound (ELBO) for Diffusion

## 一句话定义

把 diffusion 模型当作 latent-variable model（latents = $x_{1:T}$），用 Jensen 不等式得到对数似然的变分上界，从而把训练降为各时间步 KL/L2 项之和。

## 数学/技术细节

$$
-\log p_\theta(x_0) \le \mathbb{E}_q\!\left[-\log\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right] = \mathbb{E}_q\!\left[-\log p(x_T) - \sum_{t\ge 1}\log\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}\right] =: L
$$

经过 Markov 与 Bayes 重排，可拆成：

$$L = \underbrace{D_{KL}(q(x_T\mid x_0)\,\|\,p(x_T))}_{L_T,\ \text{常数}} + \sum_{t>1}\underbrace{D_{KL}(q(x_{t-1}\mid x_t,x_0)\,\|\,p_\theta(x_{t-1}\mid x_t))}_{L_{t-1}} \underbrace{- \log p_\theta(x_0\mid x_1)}_{L_0}$$

由 [[wiki/concepts/diffusion-process]]，$q(x_{t-1}\mid x_t,x_0)$ 与 $p_\theta(x_{t-1}\mid x_t)$ 都是高斯，KL 有闭式：化为 $\tilde\mu_t$ 与 $\mu_\theta$ 之间的加权 L2。

### 从 ELBO 到 L_simple

DDPM 把 $\mu_\theta$ 改写为预测 $\varepsilon$（[[wiki/concepts/epsilon-parameterization]]），并**丢掉**每个 $t$ 前的系数，得到：

$$L_\mathrm{simple} = \mathbb{E}_{t,x_0,\varepsilon}\!\left[\|\varepsilon - \varepsilon_\theta(\sqrt{\bar\alpha_t}x_0 + \sqrt{1-\bar\alpha_t}\varepsilon,\,t)\|^2\right]$$

经验上 $L_\mathrm{simple}$ 比加权 ELBO 给出更好的样本质量（NLL 略差）。**为什么更好至今没有完全令人信服的解释**——这是一个开放问题。

## 与其他概念的关系

- 拆分依赖 [[wiki/concepts/diffusion-process]] 中给出的 $q(x_{t-1}\mid x_t,x_0)$ 闭式
- 与 [[wiki/concepts/score-matching]] 的等价：$L_\mathrm{simple}$ ⇔ 加权 denoising score matching
- 后续 IDDPM (Nichol & Dhariwal 2021) 进一步学习 $\Sigma_\theta$，把 NLL 也优化上去

## 在 text-guided editing 中的作用

- 几乎所有 text-guided 工作都直接训练 $L_\mathrm{simple}$ 的条件版本 $\|\varepsilon - \varepsilon_\theta(x_t,t,c)\|^2$；ELBO 的"原型"在编辑文献中很少出现，但理解它对解读各种 reweighting / guidance 必不可少

## 出处与引用

- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §3 (eq. 3, 5, 8, 12, 14)
