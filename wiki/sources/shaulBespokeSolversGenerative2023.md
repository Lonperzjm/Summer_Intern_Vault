---
type: source
title: "Bespoke Solvers for Generative Flow Models"
aliases: [Bespoke Solver, RK2-Bespoke]
tags: [flow-matching, solver, sampling, optimization]
status: active
created: 2026-07-30
updated: 2026-07-30
raw: "[[raw/literature-notes/shaulBespokeSolversGenerative2023]]"
authors: [Neta Shaul, Juan Perez, Ricky T. Q. Chen, Ali Thabet, Albert Pumarola, Yaron Lipman]
venue: preprint (arXiv)
year: 2023
arxiv: "2310.19075"
---

# Bespoke Solvers for Generative Flow Models

## Motivation

给定预训练好的 flow model（速度场 $u_t$ 固定），通用 ODE solver（Euler、RK2）在低 NFE（5–20 步）时全局截断误差大，生成质量差。现有 dedicated solvers（DPM-Solver、DEIS 等）利用 ODE 的半线性结构做了优化，但本质上仍是基于直觉和启发式选择时间步与参数。能否**针对具体模型**自动学出最优 solver？

## Method

### 核心框架：参数化 solver 族

通过 **scale-time 变换** 定义一族参数化 solver：

$$\bar{x}(r) = s_r \cdot x(t_r), \quad r \in [0, 1]$$

其中 $t_r : [0,1] \to [0,1]$ 是时间重参数化（diffeomorphism），$\varphi_r(x) = s_r x$ 是空间缩放。变换后在 $\bar{x}$ 空间用 base solver（RK1 或 RK2）积分，再变换回原空间。

可学参数 $\theta = (\theta^t, \theta^s)$：
- $\theta^t$：时间步节点 $\{t_0, t_1, \ldots, t_n\}$ + 导数 $\{\dot{t}_i\}$
- $\theta^s$：缩放系数 $\{s_i\}$ + 导数 $\{\dot{s}_i\}$
- 总参数量极少（RK1: $4n-1$；RK2: $8n-1$；n=10 时约 80 个标量）

### 训练：RMSE 上界 loss

全局截断误差的上界：

$$e_n^\theta \leq \sum_{i=1}^n M_i^\theta d_i^\theta$$

其中 $d_i^\theta$ 是局部截断误差，$M_i^\theta = \prod_{j=i}^n L_j^\theta$ 是误差传播因子。据此定义可并行计算的 loss：

$$\mathcal{L}_{\text{bes}}(\theta) = \mathbb{E}_{x_0 \sim p(x_0)} \sum_{i=1}^n M_i^\theta \|x_{i+1}^{\text{aux}}(t_{i+1}) - x_{i+1}^\theta\|$$

训练需要 GT 轨迹（用 RK45 高精度解），成本仅为原模型训练的 ~1%。

### 关键定理

- **Theorem 2.2（Consistency）**：对任意 $\theta \in \mathcal{F}$，参数化 solver 的阶数与 base solver 相同。即步长 $h \to 0$ 时，$x_n^\theta \to x(1)$。
- **Theorem 2.3（等价性）**：搜索 scale-time 变换空间 = 搜索所有 Gaussian Path（所有 noise schedule）。这意味着 Bespoke solver 自动涵盖了 EDM schedule、cosine schedule 等所有已有的手工设计。

## Results

| 模型/数据集 | NFE | Bespoke FID | Baseline FID | GT FID |
|-----------|-----|-------------|--------------|--------|
| CIFAR-10 FM-OT | 10 | 2.73 | 4.17 (RK2) | 2.59 |
| CIFAR-10 FM-OT | 20 | 2.59 | 2.86 (RK2) | 2.59 |
| ImageNet-64 FM-OT | 10 | 2.26 | — | 1.68 |
| ImageNet-128 FM-OT | 20 | 2.45 | — | 2.30 |

- NFE=10 时比所有 dedicated solvers 改善 34%+
- NFE=20 时接近 GT（1% 以内）
- Ablation 显示 time transform 贡献大于 scale transform，但二者结合最优

## 关系

- 与 [[wiki/concepts/flow-matching]]：直接为 FM 模型设计专用 solver
- 与 DPM-Solver（Lu et al. 2022, arXiv 2206.00927；待 ingest）：Bespoke 统一覆盖 DPM-Solver 系的手工 schedule 设计，且自动找到更优解
- 与 [[wiki/sources/chaTrainingFreeRefinementFlow2026]]（FDS）：Bespoke 是全局离线优化，FDS 是在线 spatial shift——正交
- 与 [[wiki/methods/rectified-flow]]：Theorem 2.3 表明 Bespoke 搜索空间等价于搜索所有 Gaussian path

## 对我的 thesis 的启示

Bespoke 与我的"跨步"方向**不撞车**：

| 维度 | Bespoke | 我的方向 |
|------|---------|---------|
| 全局 vs 局部 | 全局固定 schedule | 局部自适应 |
| 是否感知奇异区 | 否（优化期望 RMSE） | 是（核心目标） |
| 训练需求 | 需要 GT 轨迹 | training-free |
| instance-aware | 否 | 是 |

两者可组合：Bespoke 学全局最优 base schedule → 推理时叠加局部自适应处理长尾 OOD。

**新启示**：scale 变换也是 solver 设计空间的重要维度，不只有时间步。我的框架目前只考虑 temporal 方向（缩步/跨步），spatial scale 是否也有利用空间？
