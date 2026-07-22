---
type: method
title: FreeDoM（Training-Free Energy-Guided Diffusion）
aliases: [FreeDoM, training-free energy-guided diffusion]
tags: [diffusion, guidance, energy-guidance, training-free, clean-estimate, editing]
status: stable
created: 2026-06-23
updated: 2026-06-23
sources: ["[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]"]
family: guidance
---

# FreeDoM（Training-Free Energy-Guided Diffusion）

> family `guidance`：[[wiki/concepts/training-free-guidance|training-free guidance]] 的代表方法，clean-estimate 路线（对照 [[wiki/methods/egsde|EGSDE]] 的 noisy-aligned）。

## 一句话总结

**Tweedie 估 $\hat x_0$ + 现成 clean 模型评能量 + 反传引导**：不训练任何新东西，把 CLIP/ArcFace/分割/草图/Gram/低通等现成模型当能量，评在 $\hat x_{0\mid t}$ 上，$\nabla_{x_t}E$ 修正每步采样。

## 核心机制

| 组件 | 内容 |
|---|---|
| 底座 | 任意预训练无条件扩散 / LDM（冻结） |
| clean estimate | $\hat x_{0\mid t}=\frac{1}{\sqrt{\bar\alpha_t}}(x_t+(1-\bar\alpha_t)s(x_t,t))$（Tweedie） |
| 能量 | $E(c,x_t)\approx\mathrm{Dist}(P_{\theta_1}(c),P_{\theta_2}(\hat x_{0\mid t}))$，现成模型给距离 |
| 多条件 | $E=\sum_i\eta_i D_{\theta_i}$（假设独立） |
| 引导 | $x_{t-1}=m_t-\rho_t\nabla_{x_t}E(\hat x_{0\mid t}(x_t),c)$ |
| time-travel | 仅 semantic 中段用 |
| 训练 | **无**（任务层面免训练；依赖现成无条件扩散 + 现成打分器） |

## Pipeline

```
Require: 无条件扩散 s, 现成能量 D(c, ·), 条件 c, ρ_t
for t = T down to 1:
    x̂_0 ← (x_t + (1-ᾱ_t) s(x_t,t)) / √ᾱ_t      # Tweedie clean estimate
    E   ← Dist(P(c), P(x̂_0))                    # 现成模型评在干净估计上
    g   ← ∇_{x_t} E(x̂_0(x_t), c)                # 反传过网络（含 ∂x̂_0/∂x_t）
    x_{t-1} ← m_t - ρ_t · g                      # 引导一步
    若处于 semantic 阶段: 可 time-travel 回跳重评
return x_0
```

## 适用场景与限制

**适用**：face ID / style / text / segmentation / sketch 等任意可微现成条件；可外挂 SD / ControlNet 做混合控制。

**限制**：
- **推理变贵**：每步能量反传 + time-travel 加步。
- **细结构控制弱**：ImageNet 大域、Canny 边缘即使 time-travel 也可能 poor guidance。
- **多条件独立假设**：条件冲突时结果差。
- **点估计偏差**：$\hat x_0$ 高噪声糊（[[wiki/concepts/conditional-diffusion]] §4 的 Jensen gap）。

## 关联

- 出处：[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]
- 概念：[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/energy-guidance]]、[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/classifier-guidance]]
- 最近邻：[[wiki/methods/dps|DPS]]（同构）；统一框架 TFG（待 ingest）
- **flow 版**：[[wiki/methods/fmps|FMPS]]——把本方法搬到 flow matching（用速度↔score 桥补 FM 无 score），含 gradient/free 两种 $\hat x_0$
- 对照：[[wiki/methods/egsde|EGSDE]]（noisy-aligned，须重训 $E_s$ + MC）↔ FreeDoM（clean-estimate，现成模型 + 点估计）
- 可外挂底座：[[wiki/methods/ldm]]、[[wiki/methods/controlnet]]
- 作者：Jiwen Yu、Yinhuai Wang、Chen Zhao、Bernard Ghanem、Jian Zhang（PKU + KAUST）
