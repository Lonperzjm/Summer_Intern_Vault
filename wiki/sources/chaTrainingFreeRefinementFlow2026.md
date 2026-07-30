---
type: source
title: "Training-Free Refinement of Flow Matching with Divergence-based Sampling"
aliases: [FDS, Flow Divergence Sampler, "Cha et al. 2026"]
tags: [flow-matching, inference-time, training-free, sampling, divergence, plug-and-play]
status: active
created: 2026-07-28
updated: 2026-07-28
raw: "[[raw/papers/2604.04646-training-free-refinement-flow-matching.pdf]]"
authors: [Yeonwoo Cha, Jaehoon Yoo, Semin Kim, Yunseo Park, Jinhyeon Kwon, Seunghoon Hong]
venue: ECCV 2026
year: 2026
arxiv: "2604.04646"
---

# Training-Free Refinement of Flow Matching with Divergence-based Sampling

> 源页。原文 [[raw/papers/2604.04646-training-free-refinement-flow-matching.pdf|Cha et al. 2026, ECCV 2026]]。

## Motivation

Flow Matching 学到的 marginal velocity field $u_t(x_t) = \mathbb E[v_t \mid x_t]$ 在多条 interpolant 交叉处指向 low-density 区域——和 [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]] 诊断的问题一致。但 HRF 通过改训练来建模完整速度分布，代价大。

本文问：**能否不改训练，仅在推理时修正轨迹远离高歧义区域？**

## Method：Flow Divergence Sampler (FDS)

### 核心理论（Theorem 1）

optimal CFM residual（marginal velocity 与 sample-wise velocity 间的条件 MSE）可以用 marginal velocity field 的散度（divergence）表达：

$$\mathcal{L}^\ast_\mathrm{CFM}(x_t, t) = \mathbb E\left[\|u_t(x_t) - v_t\|^2 \;\middle|\; x_t\right] = \frac{\dot\alpha_t\beta_t - \alpha_t\dot\beta_t}{\alpha_t}\left(\beta_t\nabla_{x_t}\cdot u_t(x_t) - \dot\beta_t d\right)$$

含义：**divergence 高的区域 = discrepancy 大的区域 = 速度平均化严重的区域**。而 divergence 可以纯粹从 pre-trained model $u_\theta$ 在推理时计算——不需要训练数据。

### Data-free surrogate

定义 inference-time discrepancy surrogate：
$$\hat\delta_t(x) = \nabla_x \cdot u_\theta(x, t)$$

用 Hutchinson trace estimator 高效近似（一次 forward + Jacobian-vector product）。

### Sampling：零阶局部精炼

在每个 solver step $t_k$，不直接用 $x_{t_k}$ 积分下一步，而是先做 **spatial refinement**：

1. 构造 $M$ 个候选：$x^{(m)} = x_{t_k} + \sigma_t \xi^{(m)}$，$\xi^{(m)} \sim \mathcal N(0, I)$
2. 选 divergence 最小者：$m^\ast = \arg\min_m \hat\delta_t(x^{(m)})$
3. 用 $\tilde x_{t_k} = x^{(m^\ast)}$ 代替 $x_{t_k}$ 做下一步积分

重复 $N$ 次迭代（实验中 $N=1$, $M=1$ 就够了）。

### 关键设计选择

- **零阶**（不做梯度下降）：避免 Hessian / 二阶导的高维开销，只用随机扰动 + 选最佳
- **仅在前半程**（$t < T_\mathrm{trunc} = 0.5$）做 refine：交叉/歧义集中在早期，后期收益递减
- **cosine schedule** $\sigma_t$：前期扰动大、后期衰减，优于 linear/concave
- 完全 plug-and-play：兼容 Euler / Heun / 任何 off-the-shelf solver

### 额外开销

每步 refine 只增加 $N \times M$ 次 NFE（默认 $N=M=1$ 即 +1 NFE/step）。在前半程 $T_\mathrm{trunc}$ 内生效，总额外开销 ≈ 原 NFE 的一半。实际 wall-clock 相比增加同等步数的 vanilla solver **更划算**（FDS 的 spatial correction 和 temporal refinement 正交）。

## Results

### 无条件生成

| Setting | Backbone | NFE | Baseline FID | +FDS FID |
|---------|----------|-----|-------------|----------|
| CIFAR-10 Cond | EDM | Euler 50 | 3.003 | **2.319** |
| CIFAR-10 Uncond | EDM | Euler 50 | 3.034 | **2.440** |
| ImageNet 256 | JiT-B/16 | Euler 50 | 4.151 | **3.799** |
| ImageNet 256 | JiT-L/16 | Euler 50 | 3.859 | **3.519** |
| ImageNet 256 | JiT-L/16 | Heun 99 | 2.713 | **2.496** |

Compute-matched baseline（增加等量 NFE 的 vanilla solver）无法匹配 FDS 的收益。

### vs Training-based methods（Table 4, CIFAR-10, Euler 50 NFE）

| Method | FID | IS | Params |
|--------|-----|-----|--------|
| VRFM* | 5.27 | - | 37.2M |
| HRF | 4.96 | 8.98 | 56.0M |
| EDM | 3.04 | 9.37 | 55.7M |
| **EDM + FDS** | **2.44** | **9.39** | 55.7M |

FDS 不改训练，直接超过 HRF / VRFM 等 training-based crossing-resolution 方法。

### Text-to-Image（SD3-Medium, DrawBench）

- IR / HPSv2 / Aes / CLIP 多数指标提升
- 细节保真度（时钟数字、车辆细节）显著改善

### Inverse Problems（TFG + FDS）

- Deblur FID: 64.02 → 63.17；LPIPS: 15.50 → 14.93
- SR×4 FID: 65.54 → 63.14；LPIPS: 18.70 → 16.23
- FDS 可与 TFG 等 guidance 方法叠加使用

## 关系

### 与已有 wiki 页的关联

- **[[wiki/concepts/flow-matching|Flow Matching]]**：FDS 基于 FM 的 CFM 目标推导 discrepancy 理论
- **[[wiki/methods/rectified-flow|Rectified Flow]]**：FDS 解决的问题与 RF 的路径弯曲相同，但走 inference-time refinement 路线而非 reflow/retraining
- **[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]]**：同样针对速度平均化，但 HRF 改训练（嵌套 ODE + 参数量翻倍），FDS 不改训练直接在 state space 做 spatial correction——且 FID 反超 HRF
- **[[wiki/methods/fmps|FMPS / FreeDoM 类 guidance]]**：FDS 是 guidance-free 的，它不需要外部 reward/energy，仅利用 model 自身的 divergence 信号；但两者可叠加（TFG+FDS 实验已证明）
- **Energy-guidance 线**：FDS 的 divergence surrogate 本质上是一种 **self-derived energy signal**——不依赖外部能量函数，而是从 model 内在的速度场散度中提取"哪里不可信"的信号。这与 energy-guidance 思想互补

### 对我的 thesis 的启示

**判断：高度相关。与 thesis 方向一致（inference-time, training-free improvement for flow models），且可组合。**

关键启示：
1. **Divergence 作为 discrepancy proxy**：这是一个 model-intrinsic 信号，不需要外部 energy/reward。可以考虑：在 editing 场景中，divergence 高的区域是否对应"编辑质量不确定"的区域？能否用 divergence 做 editing confidence map？
2. **Spatial refinement 与 temporal refinement 正交**：FDS 修正 state（空间），传统 solver 修正积分精度（时间）。在 editing 场景中，能否把 FDS 的 spatial refine 与 inversion/editing guidance 结合？
3. **与 energy-guidance 叠加**：FDS + TFG 已验证可叠加。如果 thesis 的 energy-guidance 方法是在推理时引入外部 energy，FDS 的 divergence 信号可以作为额外的 regularizer 或 selection criterion
4. **零阶优化 vs 梯度 guidance**：FDS 选择零阶（随机扰动+选最优）避免 Hessian，这在高维 latent（如 FLUX）中可能比梯度 guidance 更实用
5. **前半程集中干预**：与 editing 中 inversion 后前几步最关键的经验观察一致

**建议行动**：
- 在 toy experiment 中对比 FDS 的 divergence map 与 energy-guidance 的 gradient direction 是否指向相同区域
- 考虑将 divergence 作为 editing pipeline 中的 adaptive step-size / early-stopping criterion

### 个人分析：奇异点统一框架

（详见 [[research/notes/2026-07-28-singularity-unified-framework]]，延续 [[research/notes/2026-07-27-high-dim-crossing-probability|高维交叉概率分析]]）

**设定**：高维空间中两条 RF 直线轨迹最近点（奇异点 / 多模态点）距离 $d_{\min}(t)$，离散步长 $\Delta = v\delta t$，模型空间分辨率 $l$（两条路线距离 $d_{\min} < l$ 时，模型不能良好拟合分离路线，$v$ 在该尺度下被平均）。

**OOD 机制**（需两个条件同时满足）：
1. $d_{\min} < l$：两条路线距离足够近，速度场退化为平均
2. 采样时离散点恰好落入该平均区域内

$E(d_{\min})$ 均值约为 $\Delta$ 的 10–40 倍（多数采样点远离奇异区）。

**待验证假设**：$\Delta \gg l$（采样步长远大于模型空间分辨率），但 $l$ 的具体量级无法确定，需实验测量。若成立，含义是：小陷阱 + 大步长 = 容易踩进去但难以预见。

**两种 temporal 修正策略**（与 FDS 的 spatial 路线正交）：

| 策略 | 方向 | 做法 | 代表 |
|------|------|------|------|
| 缩小步长 | temporal | 检测 $v$ 变化 → 减小 $\delta t$，精细通过奇异区 | adaptive step-size solvers |
| 增大步长 | temporal | 检测 $v$ 变化 → 增大 $\delta t$，一步跨过奇异区 | （个人新思路，待验证） |
| Spatial shift | spatial | 不调步长，把 $x_t$ 推到 divergence 低的邻居 | **FDS**（本文） |

注：FDS 走 spatial 路线是否与 temporal 策略解决的是同一个问题的不同面，需要进一步确认。

**开放问题**：
- "增大步长跨过去"在 ODE 精度上有代价——是否可以跨过后做 corrector step 补偿？
- 能否用 divergence 信号同时指导 adaptive step-size（高 divergence → 缩小步长 or 增大步长的选择条件）？
- $l$ 如何实验测量？（候选：toy 2D setting 中逐步缩小路线间距，观察速度预测何时退化为平均）
- FDS 的正交性是否需要更严格论证？

## Open Questions

- [ ] FDS 在 editing 场景（RF-Inversion / SDEdit 风格）中的效果如何？divergence 高的区域是否对应编辑失真？
- [ ] 零阶 refine 与一阶 gradient guidance 在高维 latent 中的 trade-off？能否混合（前期零阶 exploration，后期一阶精确 guidance）？
- [ ] FDS 的 Hutchinson estimator 在 DiT / FLUX 级别模型上的实际 wall-clock overhead？
- [ ] divergence signal 能否替代 / 增强 CFG 的 guidance scale 选择？（高 divergence 区域需要更强 guidance？）
- [ ] FDS + energy-guidance 的组合效果——divergence 做 "where to refine"，energy 做 "how to refine"？

## 出处

- arXiv: [2604.04646](https://arxiv.org/abs/2604.04646)
- 代码: https://yeonwoo378.github.io/official_fds
- 作者: Yeonwoo Cha, Jaehoon Yoo, Semin Kim, Yunseo Park, Jinhyeon Kwon, Seunghoon Hong（KAIST）
