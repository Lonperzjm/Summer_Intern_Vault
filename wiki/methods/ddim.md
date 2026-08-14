---
type: method
title: DDIM（Denoising Diffusion Implicit Models）
aliases: [DDIM]
tags: [diffusion, sampling-acceleration, deterministic-sampling]
status: stable
created: 2026-05-14
updated: 2026-08-14
sources: ["[[wiki/sources/songDenoisingDiffusionImplicit2022]]", "[[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]"]
family: other
---

# DDIM（Denoising Diffusion Implicit Models）

> family 标作 `other`：和 [[wiki/methods/ddpm]] 一样属于**领域基础设施**，不是 text-guided editing 方法族本身。但 DDIM 的确定性采样与 inversion 能力，是下游几乎所有 inversion-based 编辑方法的底座。

## 一句话总结

复用 [[wiki/methods/ddpm|DDPM]] 训好的 ε 网络、不重训；通过把前向过程从马尔可夫松绑为**非马尔可夫族**，得到一族共享同一训练目标的生成过程，由方差参数 $\sigma$ 索引——$\sigma=0$ 即确定性的 DDIM，并可在任意时间步子序列上跳步采样，10–50× 加速。

## 核心机制

- **训练**：无。直接用 DDPM 的 $L_\text{simple}$ 训出的 $\varepsilon_\theta$（[[wiki/concepts/epsilon-parameterization]]）。Theorem 1 保证非马尔可夫族与 DDPM 共享变分目标。
- **非马尔可夫前向族**：边缘 $q_\sigma(x_t\mid x_0)=\mathcal N(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I)$ 与 DDPM 完全一致（[[wiki/concepts/non-markovian-diffusion]]）。
- **采样更新**：先从 $x_t$ 预测 $x_0$，再合成 $x_{t-1}$ ——
$$
x_{t-1} = \sqrt{\bar\alpha_{t-1}}\,\frac{x_t-\sqrt{1-\bar\alpha_t}\,\varepsilon_\theta}{\sqrt{\bar\alpha_t}} + \sqrt{1-\bar\alpha_{t-1}-\sigma_t^2}\,\varepsilon_\theta + \sigma_t z
$$
- **两个旋钮**：$\sigma_t$（$0$ → 确定性 DDIM；$\sqrt{\tilde\beta_t}$ → 退回 DDPM）；子序列 $\tau\subseteq\{1,\dots,T\}$（决定采样步数）。

## Pipeline

### Training

无独立训练流程——见 [[wiki/methods/ddpm#Training]]。

### Sampling（DDIM，$\sigma=0$，在子序列 $\tau$ 上）

```
x_T ~ N(0, I)
for i = S, S-1, …, 1:           # τ = (τ_1, …, τ_S) 是 {1,…,T} 的子序列
    t, t_prev = τ_i, τ_{i-1}    # (τ_0 := 0)
    eps = ε_θ(x_t, t)
    x0_pred = (x_t − sqrt(1 − ᾱ_t) · eps) / sqrt(ᾱ_t)
    x_{t_prev} = sqrt(ᾱ_{t_prev}) · x0_pred + sqrt(1 − ᾱ_{t_prev}) · eps
return x_0
```

### Inversion（ODE 反向，x_0 → x_T）

把上面的 ODE $\mathrm d\bar x = \varepsilon_\theta(\cdot)\,\mathrm d\sigma$ 从 $t=0$ 反向积分，可把真实图像编码回 latent $x_T$——DDIM inversion 的雏形，下游编辑方法据此实现"先 invert 再带条件 denoise"。这条 ODE 正是 [[wiki/concepts/probability-flow-ode|probability-flow ODE]] 的离散化（VP-SDE 情形），[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]] 给出其连续时间母体——可逆性、精确似然由此而来。一句话：**DDIM = diffusion 的训练 + flow 的采样**。

### 强 guidance 为什么破坏 inversion

DDIM inversion 依赖相邻步 noise prediction 近似不变。Vanilla CFG 的跨步条件误差被 $\omega$ 放大。[[wiki/methods/cfg-plus-plus|CFG++]] 用较小 $\lambda$ 形成 clean estimate，并用 unconditional prediction renoise，使对应误差从 $\omega\,\Delta\epsilon_c$ 降为 $\lambda\,\Delta\epsilon_c$。它改善 reconstruction/editing，但仍有 DDIM 离散误差。

## 适用场景与限制

**适用**：任何已训 DDPM-style 模型的快速采样；确定性采样带来的 latent 语义插值、consistency；inversion-based 图像编辑的底座。

**限制**：
- 步数极少时（< 10 步）仍有质量损失；步数–质量是平滑 trade-off 而非免费午餐。
- ODE 反向积分的离散误差会累积，inversion 的保真度受步数 / noise schedule 影响。
- 确定性使多样性下降——需要随机性时要把 $\sigma$ 调回非零。

## Failure modes

- 子序列 $\tau$ 选得太稀 → 离散化误差大、结构性 artifact。
- inversion 往返不闭合（$x_0\to x_T\to x_0'$ 偏移），尤其在强 guidance 下——是后续编辑论文反复攻击的点。

## 关联

- 概念：[[wiki/concepts/non-markovian-diffusion]]、[[wiki/concepts/epsilon-parameterization]]、[[wiki/concepts/diffusion-process]]、[[wiki/concepts/score-sde]]、[[wiki/concepts/probability-flow-ode]]、[[wiki/concepts/variational-bound-elbo]]
- 上游：[[wiki/methods/ddpm]]（共享 ε 网络与训练目标）；[[wiki/concepts/probability-flow-ode|probability-flow ODE]]（确定性采样的连续母体，[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]）
- 下游：DDIM inversion → inversion-based text-guided editing；[[wiki/methods/cfg-plus-plus|CFG++]]（降低 guidance-induced inversion error）；扩散蒸馏 / Consistency Models；"连训练也 flow 化"的近亲 → [[wiki/concepts/flow-matching|Flow Matching]] / [[wiki/methods/rectified-flow|Rectified Flow]]
- 编辑应用：**SD img2img** = [[wiki/methods/sdedit|SDEdit]] 用 DDIM 采样器的确定性版——加噪到 [[wiki/concepts/noising-strength|$t_0$（`strength`）]] 后用 DDIM reverse；"确定性 SDEdit ≈ DDIM img2img"。注意区分 **DDIM inversion**（把 $x_0$ 精确反演回 $x_T$，需要带优化的编辑方法）与 **SDEdit/img2img**（直接加噪，不反演）——同属 inversion/noising-based 派系但成本两端
- 出处：[[wiki/sources/songDenoisingDiffusionImplicit2022]]
