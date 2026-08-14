---
type: method
title: "CFG++（Manifold-Constrained Classifier-Free Guidance）"
aliases: [CFG++, manifold-constrained CFG]
tags: [diffusion, guidance, inversion, image-editing, training-free]
status: active
created: 2026-08-14
updated: 2026-08-14
sources: ["[[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]", "[[wiki/sources/galashovLearnGuideYour2025]]"]
---

# CFG++（Manifold-Constrained Classifier-Free Guidance）

## 一句话总结

只用插值 guidance 生成 clean estimate，renoising/transport 继续使用 unconditional prediction；以零额外 NFE 减少 off-manifold artifact，并提升生成与 DDIM inversion/editing 的稳定性。

## 核心更新

$$
\hat\epsilon_c^\lambda
=\hat\epsilon_\varnothing+\lambda(\hat\epsilon_c-\hat\epsilon_\varnothing),
$$
$$
\hat x_c^\lambda
=\frac{x_t-\sqrt{1-\bar\alpha_t}\hat\epsilon_c^\lambda}{\sqrt{\bar\alpha_t}},
\qquad
\boxed{x_{t-1}
=\sqrt{\bar\alpha_{t-1}}\hat x_c^\lambda
+\sqrt{1-\bar\alpha_{t-1}}\hat\epsilon_\varnothing}.
$$

与 vanilla [[wiki/concepts/classifier-free-guidance|CFG]] 的关键差异，是 renoising 项从 $\hat\epsilon_c^\omega$ 换回 $\hat\epsilon_\varnothing$：guide the denoising, not the renoising。

## Pipeline

1. 计算 unconditional prediction $\epsilon_u$。
2. 计算 conditional prediction $\epsilon_c$。
3. 形成 $\epsilon_\lambda=\epsilon_u+\lambda(\epsilon_c-\epsilon_u)$。
4. 用 $\epsilon_\lambda$ 得到 guided clean estimate。
5. 用 $\epsilon_u$ 将 clean estimate 合成下一 noisy state。

- 训练：无；复用已有 CFG 模型。
- 每步 NFE：与 CFG 相同，仍是两次。
- 主实验范围：$\lambda\in[0,1]$，不是跨 solver 的硬约束。
- 兼容：DDIM、Euler、Euler ancestral、DPM-Solver++ 2M/2S、部分 distilled models。

## 几何解释

作者从 SDS loss 与 DDS 的 manifold-constrained update 推导该式，将其解释为
$$
\text{unconditional manifold transport}
+\text{conditional optimization}.
$$
它把 base transport 和 conditional steering 分开，避免大 scale 外推整条生成方向。Manifold 论证依赖局部线性假设，不能理解为每一步都被严格投影到真实高维数据流形。

## Inversion / editing

[[wiki/methods/ddim|DDIM inversion]] 依赖跨步 noise prediction 稳定。Vanilla CFG 用 $\omega$ 放大 conditional prediction 的变化；CFG++ 的对应误差只乘通常更小的 $\lambda$，因此更接近 DDIM 原有可逆性。它仍受时间离散误差与 latent autoencoder 重建上限约束。

## 结果摘要

- SD v1.5、50-NFE DDIM、COCO 10k，匹配 $(\omega,\lambda)=(2,0.2)$ 时 FID 13.84 → 12.75，CLIP 0.298 → 0.303。
- 其余 LPIPS-matched scales 上 FID 也改善、CLIP 大体持平。
- DDIM inversion 与 real-image editing 对 scale 更稳定。
- 接入 PSLD 后，多类 inverse problem 的 FID/LPIPS 多数改善，但非所有指标都严格占优。

## 关系与边界

- [[wiki/sources/chidambaramWhatDoesGuidance2024|What Does Guidance Do?]]：解释 off-support；CFG++ 给 sampler-level mitigation。
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]]：改 timing；CFG++ 改一步轨迹。
- [[wiki/methods/learn-to-guide|Learn to Guide]]：学习 condition-/step-dependent scale；CFG++ 分解 transport 与 steering。概念上正交，尚无联合实验。
- [[wiki/concepts/non-markovian-diffusion]]：依赖确定性 DDIM 的 denoise–renoise 分解。
- Flow Matching / Rectified Flow：operator splitting 可能迁移，但本文无直接 velocity-space 公式或实证。
- $\omega\leftrightarrow\lambda$ 是配置相关的 LPIPS 匹配，不能照抄；20-NFE DPM-Solver++ 2M 的强 guidance 需要轻微 $\lambda\ge1$。
- Unconditional prediction 本身不可靠时，没有 manifold 保证。

## 出处

- [[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]
