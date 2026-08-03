---
type: source
title: "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"
aliases: [BA-solver, Bi-Anchor Solver]
tags: [flow-matching, solver, sampling, acceleration, sidenet]
status: active
created: 2026-08-02
updated: 2026-08-02
raw: "[[raw/literature-notes/chenBiAnchorInterpolationSolver2026]]"
authors: "Hongxu Chen, Hongxiang Li, Zhen Wang, Long Chen"
venue: ICML 2026
year: 2026
arxiv: "2601.21542"
---

# Bi-Anchor Interpolation Solver (BA-solver)

> Source 页。原文：[[wiki/entities/hongxu-chen|Hongxu Chen]]、Hongxiang Li、Zhen Wang、[[wiki/entities/long-chen|Long Chen]]，ICML 2026。arXiv [2601.21542](https://arxiv.org/abs/2601.21542)。

## Motivation

Flow Matching 生成需要求解 ODE $\frac{dx_t}{dt}=v_\theta(x_t,t)$，每次调用 backbone $v_\theta$ 记一次 NFE。现有方案的困境：

- **Training-free solver**（Euler、DPM-Solver、Heun 等）：大步长时外推/串行内部评估，few-step（5–10 NFE）质量显著下降
- **一两步蒸馏**（Consistency Model、MeanFlow 等）：质量好但训练代价高（优化全部 ~675M 参数、数万迭代）、不 plug-and-play

BA-solver 目标：**提高每个区间的积分精度，但不增加昂贵的 backbone NFE**。

## Method

### SideNet：轻量局部速度预测器

冻结 FM backbone，旁边加一个 ~6M 参数的 SideNet $S_\phi$（depthwise-separable conv ResBlocks + FiLM 调制）：

$$\hat v_{t+\Delta t} = v_t + \Delta t \cdot S_\phi(x_t, v_t, t, \Delta t)$$

$\Delta t$ 可正可负——**双向时间感知**（Bidirectional Temporal Perception）：从当前 anchor 向前/向后预测速度。

### 双 anchor 机制（核心）

单 anchor 时，SideNet 从固定 $x_t$ 出发预测整个区间的速度，远端 state drift 大。双 anchor 在区间两端各有一个 backbone 给出的可靠速度 $v_t,\,v_{t-h}$：

- 前半段（靠近起点）用 forward prediction
- 后半段（靠近终点）用 backward prediction

SideNet 最大预测距离从 $h$ 降为 $h/2$，误差系数减半。

### 每个区间的三阶段采样

1. **Forward Probe**：从起点 anchor $(x_t, v_t)$ 用 SideNet 预测区间内节点速度 → 求积得临时终点 $x_{t-h}^{\text{pred}}$
2. **Backward Refinement**：在临时终点调用一次 backbone 得终点 anchor $v_{t-h}=v_\theta(x_{t-h}^{\text{pred}}, t-h)$；从右端往回预测后半段节点速度
3. **重新积分**：前半段取 forward prediction，后半段取 backward prediction，高阶求积给出最终 $x_{t-h}$

### Gauss–Lobatto 求积

推理使用 4-point Gauss–Lobatto（天然包含两端点）：

$$x_{t-h} \approx x_t - \frac{h}{12}\left[v_t + 5\hat v_{\tau_2}^{\text{fwd}} + 5\hat v_{\tau_3}^{\text{bwd}} + v_{t-h}\right]$$

两个端点速度来自 backbone（精确），两个内部节点来自 SideNet（廉价）。

### Anchor Reuse

终点 anchor $v_{t-h}$ 复用为下一区间的起点 anchor → **$N$ 个区间严格 $N$ 次 backbone NFE**（最后一步用 forward-only，不多算终点）。

### 训练：Chain-based Training

连续模拟多个求解区间 $x_t \to x_{t-h} \to x_{t-2h} \to \cdots$，后一步吃前一步带误差的状态。SideNet 学会在 off-trajectory 状态下工作。

- 损失：$\mathcal{L}_\phi = \|v_{t-h}^{\text{pred}} - \text{SG}(v_{t-h})\|_2^2$（stop-gradient 不更新 backbone）
- chain length 8，batch 4096，ImageNet-256 训练 ~250 iterations

## Results

Backbone：REPA-enhanced SiT（~675M params），class-conditional ImageNet。

### 主要 FID

| NFE | ImageNet-256 | ImageNet-512 |
|-----|-------------|-------------|
| 5 | 2.84 | 5.18 |
| 7 | 1.96 | 2.88 |
| 10 | — | — |
| 15 | 1.65 | 1.83 |

7 NFE 对比：Euler 7.56 / Flow-DPM 4.03 / **BA-solver 1.96**。

### 训练效率

8 NFE FID 1.85，仅需 250 iterations、6M trainable params；对比蒸馏方法需数万迭代 + 675M params。

### 推理开销

H800 batch 256：1 次 backbone 4.18s / 4 次 SideNet 0.26s / 积分 0.01s → SideNet + 积分约 **6% 额外延迟**。

### 消融（7 NFE, ImageNet-256）

| 配置 | FID |
|------|-----|
| Bi-anchor + Gauss–Lobatto（默认） | 1.96 |
| Single-anchor | 4.35 |
| 不使用中间速度 | 3.54 |
| 改用 Simpson | 1.97 |

**结论：双 anchor >> quadrature 选择**。Gauss–Lobatto 与 Simpson 几乎无差（1.96 vs 1.97），但去掉第二个 anchor 后 FID 劣化到 4.35。

## 理论分析

- 单 anchor 局部误差 $O(h^2)$，双 anchor 把系数减半但阶数不变
- 4-point Gauss–Lobatto 对多项式≤5 精确（纯 quadrature error $O(h^7)$），但 BA-solver **总误差仍由 $O(h^2)$ 控制**（SideNet 预测误差 + state drift + 临时终点误差 + 累积误差）
- BA-solver 不是整体七阶 ODE solver；高阶求积只压低积分公式本身的误差贡献

## Image Editing

BA-solver 求解原始 FM ODE（非重训映射），保留 ODE 可逆结构。论文展示 ODE inversion + 换类别条件的编辑（Dog→Cat 等），约 10 NFE。

## 关系

### 在 [[wiki/concepts/ode-solver-taxonomy]] 中的位置

BA-solver 不完全属于既有 solver 族谱中的任何一档：

| 维度 | BA-solver 的位置 |
|------|-----------------|
| 是否训练 | 是（SideNet ~6M params，250–500 iters）——代价远低于模型蒸馏，但高于 Bespoke/BNS（<200 params） |
| 改进维度 | **Temporal 精度**：在区间内用 SideNet 做 dense velocity interpolation + 高阶求积 |
| Instance-aware | 是——SideNet 以当前 $x_t, v_t$ 为输入，每个样本独立预测 |
| 与其他 solver 正交性 | 与 [[wiki/sources/chaTrainingFreeRefinementFlow2026\|FDS]]（spatial correction）正交、可叠加；与 [[wiki/sources/shaulBespokeSolversGenerative2023\|Bespoke]]/[[wiki/sources/shaulBespokeNonStationarySolvers2024\|BNS]]（全局时间网格优化）部分正交——BA-solver 在给定时间网格下提高区间内精度 |

### 与 vault 已有方法的对比

| 方法 | 路线 | 代价 | 5–10 NFE 优势 |
|------|------|------|--------------|
| [[wiki/sources/shaulBespokeSolversGenerative2023\|Bespoke]] | 离线优化 scale-time 变换（~80 params） | 极低 | 中等（全局优化，不 instance-aware） |
| [[wiki/sources/shaulBespokeNonStationarySolvers2024\|BNS]] | 非平稳多步法（<200 params） | 极低 | 中等（严格超集 Bespoke） |
| [[wiki/sources/wangTamingRectifiedFlow2025\|RF-Solver]] | Training-free Taylor 展开 | 0 | 中等（$O(h^2)\to O(h^3)$） |
| [[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026\|FastFlow]] | Training-free bandit 跳步 | 0 | 省 NFE 而非提高精度 |
| **BA-solver** | SideNet 学区间内速度插值 | 低（6M/250 iter） | **强**（7 NFE FID 1.96，远超上述方法） |

### 关键概念链接

- [[wiki/concepts/flow-matching]]：BA-solver 求解的 ODE
- [[wiki/concepts/ode-solver-taxonomy]]：solver 谱系全景
- [[wiki/methods/rectified-flow]]：BA-solver 实验底座为 REPA+SiT（RF 变体）
- [[wiki/concepts/probability-flow-ode]]：BA-solver 保留 ODE 可逆性 → 可做 inversion/editing

### 局限性

1. **不是 training-free**：每个新 backbone / 数据集 / 条件机制需重训 SideNet
2. **实验范围集中**：仅验证 ImageNet class-conditional REPA+SiT；未验证大规模 text-to-image / 复杂 CFG
3. **NFE 不含 SideNet**：6% overhead 是 H800 batch 256 下的结果，小 batch 可能更高
4. **Anchor mismatch**：终点 anchor velocity 在 $x_{t-h}^{\text{pred}}$ 上计算，但状态随后被修正为不同的 $x_{t-h}$；下一步复用的 velocity 与真实状态不严格匹配。chain-based training 在适应这个 mismatch

## 对我的 thesis 的启示

BA-solver 在 solver taxonomy 中引入了"学习型局部 velocity interpolator"这一新范式，与 vault 已有的 solver 改进路线（全局离线优化 / training-free 高阶 / bandit 跳步）正交。对 thesis（奇异点/多模态问题 + solver 调研）的含义：

- BA-solver 展示了"冻结 backbone + 轻量 sideband 网络"在 solver 层面的价值——与 [[wiki/concepts/sideband-conditioning|sideband-conditioning]] 在条件注入层面的模式同构
- 它**不处理**速度平均化/奇异区问题（SideNet 学的是 backbone 给出的 velocity 的局部插值，如果 backbone 本身在多模态区被平均化，SideNet 也会被平均化）
- 但它的 **chain-based training 适应 off-trajectory 状态**的思路，与我的"检测到奇异区后需要在 OOD 状态上做决策"场景有潜在联系

## Open Questions

- [ ] BA-solver + CFG 的互动？论文未验证 text-to-image 场景下 CFG 是否影响 SideNet 预测质量
- [ ] 与 [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]] 叠加时，FDS 的 spatial correction 是否会改变 SideNet 的输入分布
- [ ] anchor mismatch 在更大步长（<5 NFE）下是否成为瓶颈
