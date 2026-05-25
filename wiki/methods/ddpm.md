---
type: method
title: DDPM（Denoising Diffusion Probabilistic Models）
aliases: [DDPM]
tags: [diffusion, foundational]
status: stable
created: 2026-05-10
updated: 2026-05-24
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
family: other
---

# DDPM（Denoising Diffusion Probabilistic Models）

> family 标作 `other`，因为它**不是** text-guided editing 方法族，而是**整个领域的基础设施**。所有 text-guided editing 方法都建立在 DDPM 风格训练之上。

## 一句话总结

固定的高斯加噪 forward 链 + 学习的去噪 reverse 链；网络预测注入的噪声 $\varepsilon$；用极简 L2 损失 $L_\mathrm{simple}$ 训练。

## 核心机制

- **加噪**：$q(x_t\mid x_0)=\mathcal{N}(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I)$（[[wiki/concepts/diffusion-process]]）
- **学习目标**：$L_\mathrm{simple}=\mathbb{E}\|\varepsilon-\varepsilon_\theta(x_t,t)\|^2$（[[wiki/concepts/epsilon-parameterization]] / [[wiki/concepts/variational-bound-elbo]]）
- **采样**：从 $x_T\sim\mathcal{N}(0,I)$ 起，$t=T,\dots,1$：$x_{t-1}=\frac{1}{\sqrt{\alpha_t}}(x_t-\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\varepsilon_\theta(x_t,t))+\sigma_t z$

## Pipeline

### Training

```
repeat:
    x_0 ~ data; t ~ Uniform{1,...,T}; ε ~ N(0, I)
    x_t = sqrt(ᾱ_t) x_0 + sqrt(1 - ᾱ_t) ε
    grad ← ∇θ ‖ε − ε_θ(x_t, t)‖²
    θ ← θ − lr · grad
until converged
```

### Sampling

```
x_T ~ N(0, I)
for t = T, T-1, …, 1:
    z ~ N(0, I) if t > 1 else 0
    x_{t-1} = (1/sqrt(α_t)) · (x_t − (β_t / sqrt(1 − ᾱ_t)) · ε_θ(x_t, t)) + σ_t z
return x_0
```

### 实现

- 网络：U-Net + sinusoidal timestep embedding；CIFAR-10 用 PixelCNN++ 风格；LSUN 加更多通道
- 调度：$T=1000$；$\beta_t$ 线性 $10^{-4}\to 0.02$
- 方差：$\Sigma_\theta$ 取固定 $\sigma_t^2=\beta_t$ 或 $\tilde\beta_t$，效果接近

## 适用场景与限制

**适用**：图像生成（CIFAR-10、LSUN 256×256）；后续被推广到 text-to-image、视频、3D、音频、分子。

**限制**：
- 采样慢：需 $T$ 步前向（DDPM 默认 1000 步）→ 后续 [[wiki/methods/ddim|DDIM]]/扩散蒸馏/Consistency Models 攻击此点
- NLL 略差于纯 ELBO 训练（IDDPM 通过学习 $\Sigma_\theta$ 缓解）
- 条件控制原生不支持，需 classifier guidance / [[wiki/concepts/classifier-free-guidance|classifier-free guidance]] 等扩展

## Failure modes

- 高 SNR 段（$t$ 小）的 $L_\mathrm{simple}$ 项数值条件差 → v-prediction 部分缓解
- 反向高斯近似的合法性依赖小 $\beta_t$；[[wiki/concepts/score-sde|Score SDE]] 视角下 DDPM 即 **VP-SDE（Variance Preserving）** 的离散化，连续极限使「反向高斯」严格成立，离散误差需仔细处理（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]）

## 关联

- 概念：[[wiki/concepts/diffusion-process]]、[[wiki/concepts/variational-bound-elbo]]、[[wiki/concepts/epsilon-parameterization]]、[[wiki/concepts/score-matching]]、[[wiki/concepts/langevin-dynamics]]、[[wiki/concepts/reparameterization-trick]]
- 上游：[[wiki/methods/diffusion-2015|Sohl-Dickstein 2015]]；[[wiki/methods/ncsn|Song & Ermon 2019]] (NCSN)
- 下游（直接继承 DDPM 训练形式）：[[wiki/methods/ddim|DDIM]]、IDDPM、[[wiki/concepts/score-sde|Score SDE]]、ADM (Dhariwal & Nichol)、[[wiki/concepts/classifier-free-guidance|CFG]]、Imagen、Stable Diffusion / LDM、几乎所有 text-guided editing 方法
- 路径视角：[[wiki/concepts/flow-matching|Flow Matching]] 把 DDPM 的高斯加噪路径收为其 Gaussian 条件路径族的一个特例（弯曲轨迹）；FM w/ Diffusion 复现 DDPM 结果但训练更稳，换 [[wiki/concepts/optimal-transport-path|OT 路径]]则更快（[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]）
- 出处：[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]
- 评测：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/lsun]]
