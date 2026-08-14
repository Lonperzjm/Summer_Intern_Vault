---
type: source
title: "CFG++: Manifold-Constrained Classifier-Free Guidance for Diffusion Models"
aliases: [CFG++, "Chung et al. 2025"]
tags: [diffusion, guidance, classifier-free-guidance, ddim, inversion, image-editing]
status: active
created: 2026-08-14
updated: 2026-08-14
raw: "[[raw/literature-notes/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]"
authors: "Hyungjin Chung, Jeongsol Kim, Geon Yeong Park, Hyelin Nam, Jong Chul Ye"
venue: ICLR 2025
year: 2025
arxiv: "2406.08070"
---

# CFG++: Manifold-Constrained Classifier-Free Guidance for Diffusion Models

> CFG++ 将一步采样拆成“条件引导的 clean estimate”与“无条件模型负责的 renoising / transport”：**guide the denoising, not the renoising**。它不增加 network evaluation，却改善生成质量、DDIM inversion/editing 与 text-conditioned inverse problems。

## Motivation

标准 [[wiki/concepts/classifier-free-guidance|CFG]] 使用
$$
\hat\epsilon_c^\omega
=\hat\epsilon_\varnothing+\omega(\hat\epsilon_c-\hat\epsilon_\varnothing),
\qquad \omega>1.
$$
在 DDIM 中，同一个 heavily guided prediction 同时用于 Tweedie denoising 和合成 $x_{t-1}$ 的 renoising。作者把 off-manifold 归因于两个来源：$\omega>1$ 令 clean estimate 超出 conditional prediction；renoising 再使用 $\hat\epsilon_c^\omega$，会给正确 noisy manifold 引入额外 offset。于是 high CFG 可能产生 artifact、mode collapse，并破坏 [[wiki/methods/ddim|DDIM inversion]] 的往返闭合。

这一诊断与 [[wiki/sources/chidambaramWhatDoesGuidance2024|What Does Guidance Do?]] 互补：后者从 guided PF-ODE 解释 boundary/tail concentration 与 score-error 放大；本文给出离散 sampler 层的 off-manifold 修正。

## Method

### CFG++ 更新

$$
\hat\epsilon_c^\lambda
=\hat\epsilon_\varnothing+\lambda(\hat\epsilon_c-\hat\epsilon_\varnothing),
\qquad \lambda\in[0,1],
$$
$$
\hat x_c^\lambda
=\frac{x_t-\sqrt{1-\bar\alpha_t}\hat\epsilon_c^\lambda}
{\sqrt{\bar\alpha_t}}.
$$

CFG++ 不用 guided prediction renoise，而是
$$
\boxed{
x_{t-1}^{\rm CFG++}
=\sqrt{\bar\alpha_{t-1}}\hat x_c^\lambda
+\sqrt{1-\bar\alpha_{t-1}}\hat\epsilon_\varnothing
}.
$$
Conditional signal 决定 clean estimate 往哪里走，unconditional diffusion 保留 noisy-state transport。每步仍是 conditional + unconditional 两次 forward，与 CFG 相同。

### SDS / DDS 推导

作者把 text guidance 写成 clean manifold $\mathcal M$ 上的优化：
$$
\min_{x\in\mathcal M}\ell_{\rm SDS}(x;c),\qquad
\ell_{\rm SDS}
=\|\epsilon_\theta(\sqrt{\bar\alpha_t}x+\sqrt{1-\bar\alpha_t}\epsilon,c)-\epsilon\|_2^2.
$$
将它代入 decomposed diffusion sampling（DDS）的 manifold-constrained update，可得
$$
\hat x_\varnothing+\lambda(\hat x_c-\hat x_\varnothing),
$$
随后使用 $\hat\epsilon_\varnothing$ renoise。其结构是
$$
\text{unconditional generative transport}
+\text{conditional constrained correction}.
$$

“manifold-constrained”依赖 linear / locally piecewise-linear manifold 等假设，是有理论动机且有实验支持的几何解释，不是任意真实数据 manifold 上的严格保证。

### Scale 对应与边界

SD v1.5、50-NFE DDIM 下，以同 seed 输出的 LPIPS 匹配得到：

| CFG $\omega$ | 2.0 | 5.0 | 7.5 | 9.0 | 12.5 |
|---:|---:|---:|---:|---:|---:|
| CFG++ $\lambda$ | 0.2 | 0.4 | 0.6 | 0.8 | 1.0 |

这是配置相关的经验映射，不是解析关系。20-NFE DPM-Solver++ 2M 中甚至需要轻微使用 $\lambda\ge1$ 才能获得更强 guidance，所以“CFG++ 永远只做插值”不是无条件结论。

### DDIM inversion 误差

DDIM inversion 依赖相邻步 prediction 近似不变。强 CFG 的条件分支变化被 $\omega$ 放大；CFG++ 的对应项只乘 $\lambda$：
$$
\|\varepsilon_{\rm CFG++}\|
=\lambda\|\delta\hat\epsilon_c(x_t)-\delta\hat\epsilon_c(x_{t-1})\|
<\|\varepsilon_{\rm CFG}\|.
$$
比较依赖 $\lambda<\omega$。CFG++ 更接近 DDIM 原本的可逆性，但仍保留离散化误差。

## Results

SD v1.5、50-NFE DDIM、COCO 10k，在 LPIPS 匹配的 guidance 强度下：

| $(\omega,\lambda)$ | CFG FID↓ / CLIP↑ | CFG++ FID↓ / CLIP↑ |
|---|---:|---:|
| (2.0, 0.2) | 13.84 / 0.298 | **12.75 / 0.303** |
| (5.0, 0.4) | 15.08 / 0.310 | **14.95 / 0.310** |
| (7.5, 0.6) | 17.71 / 0.312 | **17.47 / 0.312** |
| (9.0, 0.8) | 20.01 / 0.312 | **19.34 / 0.313** |
| (12.5, 1.0) | 21.23 / 0.313 | **20.88 / 0.313** |

CFG++ 在这些匹配点均降低 FID，CLIP 持平或略升；但 LPIPS matching 不是跨配置通用的公平性定义。

- DDIM inversion 在 guidance 增大时更稳定，real-image editing 更好保留背景与场景结构。
- 可接 Euler、Euler ancestral、DPM-Solver++ 2M/2S，以及 SDXL-Turbo/Lightning。
- FFHQ-512 的 PSLD inverse problems 中多数 FID/LPIPS 改善；例如 motion deblur FID 91.90 → 65.67。不同指标并非全部同步改善。

## 关系

- 核心方法：[[wiki/methods/cfg-plus-plus]]；基线是 [[wiki/concepts/classifier-free-guidance]]。
- 采样与 inversion：[[wiki/methods/ddim]] / [[wiki/concepts/non-markovian-diffusion]]。
- [[wiki/sources/chidambaramWhatDoesGuidance2024]] 解释 vanilla guidance 的 tail/off-support 行为；CFG++ 是 sampler-level mitigation。
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]] 保持更新形式、改变 timing；CFG++ 改变一步轨迹，两者正交。
- SDS / DDS 提供推导，但对应源尚未在 Vault ingest。

## 对我的 thesis 的启示

1. **不改变 working thesis 版本。** 本文未验证 Flow Matching / Rectified Flow 上的对应构造。
2. **transport 与 steering 分开是可复用设计原则。** 可研究 base transport 与 condition-induced correction 的 operator splitting，而不是外推整个 velocity。
3. **inversion failure 不只来自 solver 误差。** Guidance 形式会放大跨步 mismatch；实验应分开测 base sampler error 与 guidance-induced error。
4. **与 Reject-and-Skip 只存在方法论联系。** CFG++ 没有 detector、rollback 或远端出口搜索，不是直接先行工作或机制证据。

## 待调研方向

- [ ] CFG++ 与 Guidance Interval 组合时，scale、timing、renoising direction 是否近似正交？
- [ ] Flow Matching 中如何定义对应于 denoise / renoise 的可识别算子？
- [ ] $\lambda>1$、少步/高阶 solver 与 distilled model 下，manifold 解释的边界是什么？
- [ ] DDS / MCG 的 manifold assumptions 在 latent diffusion 中能否独立检验？

## 出处

- [[raw/literature-notes/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]
- ICLR 2025；arXiv: 2406.08070
