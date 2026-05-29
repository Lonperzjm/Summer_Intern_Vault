---
type: concept
title: Latent-Space Generative Modeling（在隐空间训生成模型）
aliases: [latent-space generative modeling, 隐空间生成建模]
tags: [latent-space, diffusion, autoencoder, paradigm]
status: stable
created: 2026-05-27
updated: 2026-05-28
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"]
---

# Latent-Space Generative Modeling（在隐空间训生成模型）

## 一句话定义

不直接在原始像素空间 $x\in\mathbb R^{3HW}$ 上训生成模型，而是先用一个 **encoder** $\mathcal E$ 把数据搬到更紧凑的 latent space $z=\mathcal E(x)\in\mathbb R^{d}$（通常 $d\ll 3HW$），在 $z$ 上训生成模型（[[wiki/methods/ddpm|diffusion]] / [[wiki/concepts/flow-matching|flow]] / GAN / autoregressive 都可以），最后用 **decoder** $\mathcal D$ 把生成的 $\hat z$ 解回像素 $\hat x=\mathcal D(\hat z)$。

## 为什么要这么做（[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] 给出的最干净论证）

像素空间的 likelihood-based 模型同时承担：

1. **perceptual compression**：去掉感知无关高频信号；
2. **semantic compression**：建模语义/概念分布。

这两件事**性质不同、最佳工具不同**：(1) 是 task-agnostic、低频/纹理任务，autoencoder + perceptual + adversarial loss 一次性预训练即可；(2) 是 task-specific、需要概率建模能力，适合扩散 / flow。在 pixel 上一锅煮 → 训练 / 采样开销爆炸。**解耦**两阶段：autoencoder 一次训，diffusion 在低维空间反复重训——是 LDM 击败 pixel diffusion 的根本原因。

## 数学/技术细节

### 通式

$$
p_\theta(x) = \int p_\theta(z)\,p(x\mid z)\,dz,\qquad
p(x\mid z) \approx \delta(x-\mathcal D(z)).
$$

- 训练阶段：$\mathcal E,\mathcal D$ 单独训成（[[wiki/concepts/perceptual-compression|perceptual compression]]），冻结；在 $\mathcal E$ 的数据分布像下训生成模型 $p_\theta(z)$。
- 采样：先 $\hat z\sim p_\theta(z)$，再 $\hat x = \mathcal D(\hat z)$。

### Trade-off：压缩率 $f$

下采样因子 $f=H/h=W/w$ 是核心旋钮：

| 情况 | 结果 |
|---|---|
| $f\to 1$ | 退化为 pixel diffusion，训练慢、采样 NFE 高 |
| $f$ 适中（$f=4,8$） | 训练 / 采样高效，autoencoder 重建误差小 |
| $f$ 过大（$\geq 16$） | autoencoder 重建上限拖累整体质量 |

### 不变量与新引入的问题

**不变**：$p_\theta(z)$ 的训练目标和采样链与 pixel 版几乎相同——LDM 在 $z$ 上跑的就是 [[wiki/methods/ddpm|DDPM]] 的训练公式。
**新引入**：
- 一个 **hard upper bound**：$\mathcal D(\mathcal E(x))\neq x$，无法被 diffusion 部分修正；
- 一个 **隐含坐标变换**：分布 $p(z)$ 的几何/拓扑取决于 $\mathcal E$ 的选择——不同 $\mathcal E$ 会给同一个 $p(x)$ 不同的 $p(z)$，扩散过程的难度也随之变。

## 与其他概念的关系

- **vs Pixel-Space Diffusion**（如 [[wiki/methods/ddpm|DDPM]]、Imagen base、Glide base）：pixel 版没有 hard upper bound，但训练 / 采样开销大一个量级；多数 pixel 版反而走"级联超分"（low-res diffusion + super-res diffusion）的工程妥协，本质也是分工。
- **vs VAE**：经典 VAE 也是 latent generative model，但强 KL 让 latent 太接近高斯，重建模糊；latent diffusion 用**弱正则 autoencoder** 保留细节、把"复杂 prior"的建模交给 diffusion，避开 VAE 的模糊问题。
- **vs LAtent Flow / LDM**：本概念是范式 / 抽象层；具体落地见 [[wiki/methods/ldm|LDM]]（latent diffusion）、SD3/FLUX（latent flow matching / rectified flow）、VQ-Diffusion / MaskGIT（latent autoregressive on VQ codes）。
- **与 [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] / [[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 的正交性**：score / flow 的理论框架在 $z$ 空间继续适用——LDM 没有挑战 score / flow，它只是改变了"score / flow 作用的空间"。

## 在 text-guided editing 中的作用

- **几乎所有现代 text-guided editing 都默认在 latent 空间做**：inversion、attention injection、[[wiki/methods/controlnet|ControlNet]] sideband 注入都发生在 $z$ 上；像素 $x$ 仅在 input / output 出现。
- 这给 thesis 的 fidelity↔controllability trade-off 加了一个**正交维度**：当前 working thesis 把该旋钮形式化为"在哪个 $t$ 注入条件 + 多强"；latent 空间引入的"**在被注入之前数据已经被压缩到什么程度**"是该旋钮**正交且更上游**的变量（见 [[wiki/concepts/perceptual-compression]]）。

## 出处与引用

- 主要出处：[[wiki/sources/rombachHighResolutionImageSynthesis2022]] §3.1
- 上游：VQGAN（Esser et al. 2021）、VQ-VAE（van den Oord et al. 2017）—— "在 VQ codes 上训 transformer" 是 latent-space generative modeling 的早期成功案例，但走 autoregressive 而非 diffusion
- 下游：[[wiki/methods/ldm]]，SD1.x/2.x / SD3 / FLUX（仍待 ingest）
