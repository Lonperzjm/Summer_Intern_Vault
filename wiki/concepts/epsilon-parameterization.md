---
type: concept
title: ε-prediction（噪声预测参数化）
aliases: [epsilon prediction, noise prediction, ε-pred]
tags: [diffusion, training-objective, parameterization]
status: stable
created: 2026-05-10
updated: 2026-05-10
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# ε-prediction（噪声预测参数化）

## 一句话定义

让 diffusion 网络的输出从"预测 reverse 高斯均值 $\mu_\theta$"改为"**预测 forward 时注入的噪声 $\varepsilon$**"，等价但训练显著更稳、质量更好——这是 DDPM 把 diffusion 推上 SOTA 的关键工程动作。

## 数学/技术细节

由 [[wiki/concepts/diffusion-process]]：$x_t = \sqrt{\bar\alpha_t}\,x_0 + \sqrt{1-\bar\alpha_t}\,\varepsilon$。把 reverse 均值重写为

$$
\mu_\theta(x_t,t) = \frac{1}{\sqrt{\alpha_t}}\!\left(x_t - \frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\,\varepsilon_\theta(x_t,t)\right)
$$

代入 [[wiki/concepts/variational-bound-elbo]] 的 $L_{t-1}$ 项，化简后得：

$$L_{t-1} - C = \mathbb{E}_{x_0,\varepsilon}\!\left[\frac{\beta_t^2}{2\sigma_t^2 \alpha_t (1-\bar\alpha_t)}\,\|\varepsilon - \varepsilon_\theta(\sqrt{\bar\alpha_t}x_0+\sqrt{1-\bar\alpha_t}\varepsilon,\,t)\|^2\right]$$

去掉 $t$ 相关权重就是 $L_\mathrm{simple}$。

### 三种等价参数化

| 预测目标 | 形式 | 备注 |
|---|---|---|
| $\mu_\theta(x_t,t)$ | 直接预测 reverse 均值 | DDPM 之前的默认；训练数值不稳 |
| $\varepsilon_\theta(x_t,t)$ | 预测注入噪声 | **DDPM 的选择**；与 score matching 等价 |
| $x_0$（"data prediction"） | 预测原图 | DDIM、v-prediction 系常见 |
| $v_\theta = \sqrt{\bar\alpha_t}\varepsilon - \sqrt{1-\bar\alpha_t}x_0$ | "v-prediction" | Salimans & Ho 2022；高 SNR 段更稳 |

三者通过 $x_t,\bar\alpha_t$ 互推。**选择哪一个**会影响不同噪声尺度上的 loss 权重和数值条件数，进而影响最终质量。

## 与其他概念的关系

- 与 [[wiki/concepts/score-matching]] 的等价：$\varepsilon_\theta(x_t,t) \approx -\sqrt{1-\bar\alpha_t}\,\nabla_{x_t}\log p(x_t)$
- 是 [[wiki/concepts/variational-bound-elbo]] → $L_\mathrm{simple}$ 化简的载体
- 是 [[wiki/methods/ddpm]] 的核心组件

## 在 text-guided editing 中的作用

- **事实标准**：从 Imagen / Stable Diffusion 到 InstructPix2Pix，几乎所有现代 text-guided diffusion 都训练 ε-pred；编辑算法的差异主要在条件注入、inversion、guidance，而非底层目标
- 因此 thesis 可以默认 ε-pred 为基础设施层

## 出处与引用

- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]] §3.2（重参数化推导）、§3.4（$L_\mathrm{simple}$）
- v-prediction 出处：Salimans & Ho 2022（"Progressive Distillation"）
