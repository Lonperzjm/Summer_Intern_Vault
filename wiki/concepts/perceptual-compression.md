---
type: concept
title: Perceptual Compression（感知压缩）
aliases: [perceptual compression, 感知压缩]
tags: [autoencoder, latent-space, lpips, vqgan, diffusion]
status: stable
created: 2026-05-27
updated: 2026-05-27
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"]
---

# Perceptual Compression（感知压缩）

## 一句话定义

**感知压缩**指：用一个自编码器 $(\mathcal E,\mathcal D)$ 把图像 $x$ 编码成低维 $z=\mathcal E(x)$，使 $\mathcal D(z)$ 在**感知层面**与 $x$ 几乎不可分辨（人眼无差），但**像素层面**可以有差距——即专门去掉感知无关的高频细节、保留语义/纹理结构信息。这一概念在 [[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] 中被用作扩散模型的**预处理层**，与 "semantic compression"（扩散自己负责）解耦。

## 数学/技术细节

不依赖单一损失，而是**三类 loss 协同**：

$$
\mathcal L_{\mathrm{AE}}(x) = \underbrace{\|\mathcal D(\mathcal E(x)) - x\|_1}_{\text{recon (L1)}}
+ \lambda_p\,\underbrace{\mathrm{LPIPS}(\mathcal D(\mathcal E(x)),\,x)}_{\text{perceptual}}
+ \lambda_{\mathrm{adv}}\,\underbrace{\mathcal L_{\mathrm{GAN}}(\mathcal D(\mathcal E(x)))}_{\text{patch-discriminator}}
+ \lambda_{\mathrm{reg}}\,\mathcal R(\mathcal E(x)).
$$

- **Reconstruction (L1)** 提供像素层面下界；
- **LPIPS perceptual** 在 VGG 特征空间衡量"看起来像不像"；这是把"感知"显式注入压缩目标的核心；
- **Adversarial**（patch discriminator）让重建更"真实"，避免 L1+LPIPS 常见的模糊；
- **正则** $\mathcal R$ 控制 latent 分布：
  - **KL-reg**：$\mathrm{KL}(q(z\mid x)\|\mathcal N(0,I))$，弱权重（不像 VAE 那么强）——SD 用此。
  - **VQ-reg**：把 latent 离散化到 codebook（VQGAN 风格），但量化算子放进 decoder 内、diffusion 仍作用于连续 latent。

### 与"semantic compression"的分工（[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] §3.1）

| 阶段 | 负责什么 | 训练对象 | 训练频率 |
|---|---|---|---|
| **perceptual compression** | 去掉感知无关高频信号 | autoencoder $(\mathcal E,\mathcal D)$ | **一次性**，多任务复用 |
| **semantic compression** | 建模语义 / 概念层分布 | diffusion U-Net $\varepsilon_\theta$ | 每个任务重训 |

LDM 的核心论点正是这条解耦——pixel diffusion 同时承担两件事是不划算的，因为 (1) 是 task-agnostic、可一次性预训练。

## 与其他概念的关系

- 是 [[wiki/methods/ldm|LDM]] / Stable Diffusion 的**前置层**；diffusion 不动，但 $\mathcal E,\mathcal D$ 是新引入的组件。
- **上游**：VQGAN（Esser et al. 2021，CompVis 前作）—— LDM 的 autoencoder 即从 VQGAN 演化（替换 VQ-reg 为 KL-reg 即得 SD 用的 VAE）。
- **与 VAE 区别**：标准 VAE 的 KL 权重高，latent 强趋近 $\mathcal N(0,I)$，重建模糊；perceptual compression 用**弱 KL + 强重建/感知/对抗**，更接近"几乎无损的低维表示" + 弱正则。
- **限制**：$\mathcal D(\mathcal E(x))\neq x$ 永远成立——这是 LDM 上所有任务的 **hard upper bound**，对像素级精确任务（细小文字、人脸 micro-feature、高频纹理）尤其敏感。

## 在 text-guided editing 中的作用

- **上游 fidelity 上界**：所有 SD 上的 text-guided editing 都共享此上界——即使条件注入与采样器都最优，autoencoder 抹除的细节也无法被恢复。
- **inversion 的隐含污染**：DDIM inversion 等都在 latent 上做往返，重建评估指标 $\|\mathcal D(\hat z_0)-x\|$ 本身已被 $\mathcal E,\mathcal D$ 的有损性污染——把"inversion 失败"和"autoencoder 失真"解耦是 thesis 可立的一个诊断维度。
- **与 fidelity↔controllability trade-off 正交**：[[wiki/overview]] 推论 2 把该 trade-off 形式化为"在哪个 $t$ 注入条件 + 多强"，而 perceptual compression 是**更上游、独立于 $t$ 的** fidelity 上界。

## 出处与引用

- 主要出处：[[wiki/sources/rombachHighResolutionImageSynthesis2022]] §3.1（Perceptual Image Compression）、§4.1（$f$ 的 ablation）
- 上游：VQGAN (Esser et al. 2021, "Taming Transformers"，仍待 ingest)
- LPIPS：Zhang et al. 2018
