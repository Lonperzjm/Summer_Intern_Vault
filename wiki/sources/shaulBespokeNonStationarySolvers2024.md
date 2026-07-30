---
type: source
title: "Bespoke Non-Stationary Solvers for Fast Sampling of Diffusion and Flow Models"
aliases: [BNS Solver, Non-Stationary Solver, "Shaul et al. 2024"]
tags: [flow-matching, diffusion, solver, sampling, distillation, ode]
status: active
created: 2026-07-30
updated: 2026-07-30
raw: "[[raw/literature-notes/shaulBespokeNonStationarySolvers2024]]"
authors: [Neta Shaul, Uriel Singer, Ricky T. Q. Chen, Matthew Le, Ali Thabet, Albert Pumarola, Yaron Lipman]
venue: ICML 2024
year: 2024
arxiv: "2403.01329"
---

# Bespoke Non-Stationary Solvers for Fast Sampling of Diffusion and Flow Models

> 源页。原文 [[raw/literature-notes/shaulBespokeNonStationarySolvers2024|Shaul et al. 2024, ICML 2024]]。
> 前作：[[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solvers (2023)]]（scale-time 变换族）。

## Motivation

[[wiki/sources/shaulBespokeSolversGenerative2023|前作]] 通过 scale-time 变换参数化 solver，但本质上仍受限于 base solver 的结构（RK1/RK2）——每步更新只能用当前步的速度评估。能否设计一个**严格更大的 solver 族**，允许每步利用所有历史信息，从而在相同 NFE 下进一步逼近高精度 teacher？

核心思想一句话：**不蒸馏生成模型，而是针对一个已训练好的 diffusion/flow 模型，学习一个专用的 ODE 数值求解器。**

## Method

### Non-Stationary Solver 族

传统方法每步使用相同形式的更新规则（stationary）。NS solver 打破这个限制：第 $i$ 步可以使用之前**所有**的状态和速度评估：

$$x_{i+1} = a_i \, x_0 + \sum_{j=0}^{i} b_{ij} \, u_{t_j}(x_j)$$

同时学习每次调用模型的时间点：

$$0 = t_0 < t_1 < \cdots < t_n = 1$$

对于每一步 $i$，系数 $(a_i, b_{i0}, \ldots, b_{ii})$ 都可以不同——这就是"non-stationary"的含义：**不同时间步不共享同一更新规则**。

直观上，这是一个可学习的、非平稳的全历史 multistep solver。

### 参数量

$n$ 步 BNS solver 的总参数量：

$$p = \frac{n(n+5)}{2} + 1$$

例如 $n=16$ 时 $p=169$。即使生成模型有几十亿参数，solver 也只有不到 200 个标量。

### Solver Taxonomy（理论贡献）

论文证明了严格的包含关系：

$$\text{RK / Exponential-RK} \subset \text{Scale-Time RK} \subset \text{Non-Stationary Solvers}$$
$$\text{Multistep / Exponential-Multistep} \subset \text{Scale-Time Multistep} \subset \text{Non-Stationary Solvers}$$

因此 Euler、RK、DDIM、DPM-Solver、指数积分器、带 scheduler/time reparameterization 的方法（包括[[wiki/sources/shaulBespokeSolversGenerative2023|前作的 scale-time 族]]）都是 NS solver 参数空间中的特殊情况。BNS 不是在某个固定 solver 上调步长，而是在一个**严格更大的求解器族**里搜索。

### 训练

对于冻结的预训练模型 $u_t$：

1. 生成 teacher trajectory：从噪声 $x_0$ 用 adaptive RK45 积分到 $x^\star(1)$（只需 520 对）
2. 用低 NFE 的 NS solver 得到 $x_n^\theta$
3. 优化 solver 参数 $\theta$：

$$\mathcal{L}(\theta) = -\mathbb{E} \log \|x_n^\theta - x^\star(1)\|^2$$

训练直接对整个采样过程反向传播。通常从 Euler 或 RK-Midpoint 初始化。高 CFG 场景下通过 scheduler/scale-time transformation 做预条件。

### 速度场统一形式

针对 Gaussian probability path 的模型，速度场统一写成：

$$u_t(x) = \beta_t \, x + \gamma_t \, f_t(x)$$

系数 $\beta_t, \gamma_t$ 取决于 scheduler 和 model type（$\epsilon$-prediction / $x$-prediction / velocity-matching）。BNS 通过学习如何组合各步速度评估 $U_i$ 来最小化离散化误差。

## Results

### ImageNet 类条件生成

| Setting | NFE | Baseline PSNR | BNS PSNR | BNS FID |
|---------|-----|---------------|-----------|---------|
| ImageNet-64 | 16 | 16.28 (RK-Mid) | **45** | **1.76** |
| ImageNet-64 | 8 | — | — | 3.90 |
| ImageNet-64 | 16 | — | — | 2.62 |

16 NFE 时比次优方法（DPM++, DDIM 等）高约 5–10 dB PSNR。

### Text-to-Image（22B 参数 latent FM-OT 模型）

| CFG scale | NFE | RK-Midpoint PSNR | BNS PSNR |
|-----------|-----|-------------------|-----------|
| $w=2.0$ | 16 | 16.28 | **29.13** |
| $w=6.5$ | 16 | 9.98 | **21.23** |

PickScore 也有改善。高 CFG 下 PSNR 提升更大（因为 CFG 放大了截断误差）。

### 与 Progressive Distillation 对比（ImageNet-64）

| NFE | PD FID | BNS FID | BNS GPU forwards | 训练样本 |
|-----|--------|---------|------------------|---------|
| 4 | **4.79** | 31.83 | — | — |
| 8 | **3.39** | 3.90 | 4.9M | 520 |
| 16 | 2.97 | **2.62** | 9.7M | 520 |

- **1–4 NFE：模型蒸馏明显更强**
- **8–16 NFE：BNS 接近甚至超过 PD**
- BNS 训练成本仅为 PD 的 ~0.5% GPU forwards

### 音频生成

在 LibriSpeech TTS 和 AudioCaps 上，BNS 相比 Euler/RK-Midpoint 和前作 BST 提高约 1–3 dB SNR。

## 注意事项

BNS 优化的主要目标是 **trajectory fidelity**（低 NFE 复现高精度 teacher 输出），用 PSNR/SNR 衡量。这不必然意味着 FID/CLIP 等分布指标同比改善——某些配置下 BNS 的 PSNR 大幅提升但 FID 并未优于普通 RK。理解结论时须区分这两类目标。

## 局限性

1. **不同 NFE 需单独训练一个 solver**（8-step 和 16-step 不能共享）
2. **Model-specific**：模型、scheduler 或 CFG scale 改变后需重新优化
3. **1–4 NFE 极低步数区域明显落后于模型蒸馏**
4. Text-to-image 实验使用私有 22B 模型，复现性有限
5. 仍依赖 CFG，一次 NFE 的实际计算量可能包含 conditional/unconditional 两次 forward

## 关系

### 与已有 wiki 页的关联

- **[[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solver 2023]]**：前作，用 scale-time 变换参数化（~80 params）。本文将搜索空间从 scale-time 族扩展到 NS 族（严格超集），参数量从 $4n-1$ 增加到 $n(n+5)/2+1$，效果全面超越
- **[[wiki/concepts/flow-matching|Flow Matching]]**：BNS 对 FM 和 diffusion 模型通用，速度场统一为 $u_t = \beta_t x + \gamma_t f_t$
- **[[wiki/methods/rectified-flow|Rectified Flow]]**：RF 通过 reflow 把轨迹拉直以适配简单 solver；BNS 反其道：保持轨迹不变，让 solver 适配轨迹
- **[[wiki/concepts/probability-flow-ode|PF-ODE]]**：BNS 的 solver taxonomy 统一覆盖了 PF-ODE 上的所有经典积分方案
- **[[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]]**：BNS 是全局离线优化（temporal），FDS 是在线 spatial shift——正交；可组合

### 对我的 thesis 的启示

**判断：高度相关。BNS 是你方向的重要 baseline——同属 solver 改进路线，但解决不同层面的问题。**

| 维度 | BNS | 我的方向（temporal adaptive） |
|------|-----|------|
| 全局 vs 局部 | 全局固定 schedule（per-model） | per-sample / per-region 自适应 |
| 是否感知奇异区 | 否（优化期望 RMSE，不区分奇异/平坦区） | 是（核心目标：检测交叉→调整策略） |
| 训练需求 | 需要 GT 轨迹（520 对 + 离线优化） | training-free（目标） |
| instance-aware | 否（同一 solver 用于所有样本） | 是 |
| 极低 NFE（1–4） | 差 | 不是主战场 |

**关键区分点**：BNS 假设速度场 $u_t$ 本身是准确的，只优化离散化方案。你的问题是速度场在交叉区域**本身就有系统性 bias**（被平均化），此时即使 solver 完美复现 ODE 轨迹，轨迹本身就是错的。这是 BNS 无法解决的——你的 novelty 空间在这里。

**可组合方向**：BNS 学全局最优 base schedule → 推理时叠加你的局部自适应处理长尾 OOD case。

## Open Questions

- [ ] BNS 的训练过程中，那 520 对 teacher trajectory 是否包含了经过奇异区的样本？如果是，BNS 是否在期望意义上已经部分缓解了奇异区的问题？
- [ ] 能否把 BNS 的优化框架改为 instance-aware：不学固定系数，而是让系数依赖于当前状态 $x_i$（变成一个极轻量的 MLP）？这是否已有人做了？
- [ ] BNS + adaptive step-size 的组合是否有价值：BNS 提供好的初始 schedule，adaptive 在其基础上微调？
- [ ] 与 [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]] 叠加：BNS（temporal optimality）+ FDS（spatial refinement）是否能进一步压缩 NFE？

## 出处

- arXiv: [2403.01329](https://arxiv.org/abs/2403.01329)
- 会议: ICML 2024
- 作者: Neta Shaul, Uriel Singer, Ricky T. Q. Chen, Matthew Le, Ali Thabet, Albert Pumarola, Yaron Lipman（Meta FAIR / Weizmann Institute）
