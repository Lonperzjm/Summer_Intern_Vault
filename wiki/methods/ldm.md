---
type: method
title: LDM（Latent Diffusion Models）
aliases: [LDM, Latent Diffusion, Stable Diffusion 前身, LDM-KL-8]
tags: [diffusion, latent-space, autoencoder, cross-attention, foundational]
status: stable
created: 2026-05-27
updated: 2026-05-28
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"]
family: other
---

# LDM（Latent Diffusion Models）

> family 标作 `other`：LDM **不是** text-guided editing 方法族本身，而是几乎所有现代 text-to-image / text-guided editing 方法的**底座**。从 [[wiki/entities/stable-diffusion|SD1.x/2.x]] 到 InstructPix2Pix、[[wiki/methods/controlnet|ControlNet]]、Prompt2Prompt 等编辑系，统统建在 LDM 管线之上。

## 一句话总结

把 diffusion 从像素空间搬到**预训练感知自编码器**给出的低维 latent $z=\mathcal E(x)$ 上做——训练 / 采样代价大幅下降，**训练目标与采样链几乎不动**；条件用 [[wiki/concepts/cross-attention|cross-attention]] 注入 U-Net（token 类条件），或直接 concat 到 noisy latent（空间对齐条件）。

## 核心机制

### 两阶段管线

```
阶段 1（一次性，多任务复用）：
    Autoencoder  E(·), D(·)
    Loss = recon + perceptual(LPIPS) + adversarial + reg(KL 或 VQ)

阶段 2（任务相关）：
    在 z = E(x) 上训 ε-pred 扩散：
        L_LDM = E ‖ε − ε_θ(z_t, t, τ_θ(y))‖²
    采样：DDIM / DDPM / PF-ODE 任选，得 ẑ_0 → D(ẑ_0) → x̂
```

### 关键组件

- **自编码器**：下采样因子 $f\in\{1,2,4,8,16,32\}$；正则两种——**KL-reg**（VAE 式，弱 KL 拉向 $\mathcal N(0,I)$，SD 用）/ **VQ-reg**（VQ codebook，但量化算子放进 decoder 内，diffusion 仍作用于连续 latent）。Loss = reconstruction + perceptual(LPIPS) + patch-discriminator adversarial。
- **Diffusion 模型**：U-Net + sinusoidal timestep embedding；几乎照搬 [[wiki/methods/ddpm|DDPM]] 训练（[[wiki/concepts/epsilon-parameterization|ε-prediction]]、[[wiki/concepts/diffusion-process|高斯加噪]]、[[wiki/concepts/variational-bound-elbo|ELBO]] 同构）。**唯一差异**：把 $x$ 换成 $z$。
- **条件编码器 $\tau_\theta$**：把任意模态 $y$ 编码成 token 序列 $\tau_\theta(y)\in\mathbb R^{M\times d_\tau}$。文本走 BERT-style transformer（SD 后来换成 CLIP 文本编码器）；layout / class 等也都 token 化。
- **条件注入两通道**：
  1. **Cross-attention**（token 类条件）：$Q$ 来自 U-Net 空间 feature、$K,V$ 来自 $\tau_\theta(y)$（详见 [[wiki/concepts/cross-attention]]）
  2. **Concat**（空间对齐条件，如 semantic map / 低清图 / mask）：直接拼到 noisy latent 的通道维 → 借助 U-Net 全卷积特性可外推到训练分辨率以外。

## Pipeline

### Training

```
# 阶段 1（autoencoder，一次性）
repeat:
    x ~ data
    z, x̂ = E(x), D(E(x))
    L_AE = L_recon(x̂, x) + λ_p · LPIPS(x̂, x)
          + λ_adv · L_GAN(x̂)
          + λ_reg · {KL(E(x)‖N(0,I)) 或 VQ codebook loss}
    update {E, D, Discriminator}
freeze E, D

# 阶段 2（latent diffusion）
repeat:
    x ~ data, y ~ condition; t ~ U{1,...,T}; ε ~ N(0, I)
    z = E(x)                         # freeze
    z_t = sqrt(ᾱ_t) · z + sqrt(1 - ᾱ_t) · ε
    c = τ_θ(y)                       # condition tokens
    L = ‖ε − ε_θ(z_t, t, c)‖²        # cross-attn 把 c 注入 U-Net
    update {ε_θ, τ_θ}                # τ_θ 可一起训也可冻结预训练
```

### Sampling（DDIM 为主，与 pixel 空间同公式只是改字母）

```
z_T ~ N(0, I)
c = τ_θ(y)
for t = T, T-Δ, …, 1:
    [optional CFG] ε̂ = ε_θ(z_t, t, ∅) + s · (ε_θ(z_t, t, c) − ε_θ(z_t, t, ∅))
    z_{t-Δ} = DDIM-update(z_t, ε̂, t)
return D(z_0)
```

### 实现要点

- $f=8$（**LDM-KL-8**）是甜点；SD1.x/2.x 即此配置。
- Latent 归一化：训练前对 $\mathcal E(x)$ 做 component-wise std rescaling（SD 中即 `scaling_factor=0.18215`）。
- 推断时 [[wiki/concepts/classifier-free-guidance|CFG]] 与 cross-attention 注入**联用**：cross-attention 把 $y$ 作为 conditional drift，CFG 在采样期把 conditional/unconditional ε 线性外推放大方向。两者正交。

## 适用场景与限制

**适用**：
- text-to-image（SD1.x/2.x）
- class-conditional / layout-to-image / semantic-to-image / super-resolution / inpainting（同一管线、改 $\tau_\theta$ 与注入通道）
- text-guided **editing** 的默认底座，覆盖 [[wiki/overview]] 主要派系前四类：inversion-based / attention-injection / [[wiki/methods/controlnet|control/sideband-injection]] / instruction-tuned（第 5 类 flow-matching-based 也以 LDM 风格的 latent 管线为基础，但换训练目标）

**限制**：
- **autoencoder 重建上限**：$\mathcal D(\mathcal E(x))\neq x$ 是 hard upper bound——对像素级精确任务（细小文字、人脸 micro-feature、高频纹理）会失真，并污染编辑保真度评测。
- **inversion 在 latent 上往返**：DDIM inversion 的失败模式可能与 autoencoder 重建误差耦合，不易解耦。
- 仍是多步采样（虽比 pixel diffusion 快 5–10×），相对 GAN 慢；后续 [[wiki/methods/rectified-flow|RF]] / 蒸馏在此基础上进一步压缩。
- 训练目标仍是 ε-pred，未享受 [[wiki/concepts/flow-matching|FM]] / RF 的"恒定 NFE"优势——SD3 / FLUX 即把 LDM 管线 ⊕ RF 训练目标组合的产物。

## Failure modes

- 高频细节 / 细小文字渲染差（autoencoder 抹除）
- VQ-reg vs KL-reg 在编辑任务上的差异未充分研究
- Cross-attention map 在某些 prompt 上"漏注意力"（prompt 中某些 token 不被任何空间位置选中）—— Prompt2Prompt / Attend-and-Excite 等编辑方法即攻击此点

## 关联

- 出处：[[wiki/sources/rombachHighResolutionImageSynthesis2022]]
- 概念基础：[[wiki/concepts/perceptual-compression]]、[[wiki/concepts/latent-space-generative-modeling]]、[[wiki/concepts/cross-attention]]、[[wiki/concepts/epsilon-parameterization]]、[[wiki/concepts/diffusion-process]]
- 上游训练范式：[[wiki/methods/ddpm|DDPM]]（ε-pred 与 $L_\mathrm{simple}$）、[[wiki/methods/ddim|DDIM]]（默认采样器）；[[wiki/concepts/classifier-free-guidance|CFG]]（推断期标配）
- 连续视角：[[wiki/concepts/score-sde|Score SDE]] / [[wiki/concepts/probability-flow-ode|PF-ODE]] 在 $z$ 空间继续成立——SD 上的 PF-ODE 采样器与 DDIM inversion 是此一致性的直接利用
- 正交线（训练目标）：[[wiki/concepts/flow-matching|Flow Matching]] / [[wiki/methods/rectified-flow|Rectified Flow]] 改训练目标但**不动**压缩层；SD3 / FLUX = LDM 压缩层 ⊕ RF/FM 训练目标
- 人物 / 机构：[[wiki/entities/robin-rombach]]、[[wiki/entities/bjorn-ommer]]、[[wiki/entities/compvis]]、[[wiki/entities/lmu-munich]]
- 具名落地权重：[[wiki/entities/stable-diffusion|Stable Diffusion]]（SD1.x / 2.x / SDXL）—— 几乎所有 text-guided editing 论文的事实底座
- 下游编辑方法：✅ [[wiki/methods/controlnet|ControlNet]]（已 ingest 2026-05-28，附着于 SD1.5；sideband + zero-conv 注入空间条件）；其余仍待 ingest：Prompt-to-Prompt、Null-text Inversion、InstructPix2Pix、T2I-Adapter、IP-Adapter、Plug-and-Play、MasaCtrl、StyleAligned …
