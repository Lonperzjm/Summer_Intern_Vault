---
type: source
title: "Taming Rectified Flow for Inversion and Editing"
aliases: [RF-Solver, RF-Edit, "Wang et al. 2025"]
tags: [rectified-flow, solver, inversion, editing, training-free, ode, taylor-expansion]
status: active
created: 2026-07-30
updated: 2026-07-30
raw: "[[raw/literature-notes/wangTamingRectifiedFlow2025]]"
authors: [Jiangshan Wang, Junfu Pu, Zhongang Qi, Jiayi Guo, Yue Ma, Nisha Huang, Yuxin Chen, Xiu Li, Ying Shan]
venue: preprint (arXiv)
year: 2025
arxiv: "2411.04746"
---

# Taming Rectified Flow for Inversion and Editing

> 源页。原文 [[raw/literature-notes/wangTamingRectifiedFlow2025|Wang et al. 2025]]。

## Motivation

基于 Rectified Flow 的模型（FLUX、OpenSora）生成能力很强，但 **inversion 精度差**：从真实图像反推噪声时，由于 ODE 离散化误差在每步累积，反推得到的 latent 无法精确重建原图。这直接制约了下游的 editing 质量——编辑流程依赖"先 inversion 再 denoise with new prompt"，如果 inversion 本身不准，编辑结果会丢失结构。

已有方法（ReNoise、RF-Prior、Rout et al.）通过增加优化步骤或额外网络来弥补，但要么需要训练，要么推理时间代价大。

本文问：**能否从 ODE 求解本身入手，用更高阶的局部近似，在不引入任何训练的前提下把 inversion 误差压下去？**

## Method

### RF-Solver：高阶 Taylor 展开求解 RF ODE

Rectified Flow 的 ODE：

$$\frac{dZ_t}{dt} = v_\theta(Z_t, t)$$

从 $t_i$ 到 $t_{i-1}$ 的精确积分形式（variation of constants）：

$$Z_{t_{i-1}} = Z_{t_i} + \int_{t_i}^{t_{i-1}} v_\theta(Z_\tau, \tau) \, d\tau$$

vanilla Euler 直接用 $v_\theta(Z_{t_i}, t_i)$ 近似整个积分——每步误差 $O(h^2)$。

RF-Solver 对速度场做 **高阶 Taylor 展开**：

$$v_\theta(Z_\tau, \tau) = \sum_{k=0}^{n-1} \frac{(\tau - t_i)^k}{k!} v_\theta^{(k)}(Z_{t_i}, t_i) + O((\tau - t_i)^n)$$

取二阶（$n=2$，ablation 显示最优）：

$$Z_{t_{i-1}} = Z_{t_i} + (t_{i-1} - t_i) \, v_\theta(Z_{t_i}, t_i) + \frac{1}{2}(t_{i-1} - t_i)^2 \, v_\theta^{(1)}(Z_{t_i}, t_i)$$

其中一阶导数用有限差分估计：在 $t_i + \Delta t$（$\Delta t = 0.01$）处再做一次 forward，差分得到 $v_\theta^{(1)}$。

**误差从 $O(h^2)$ 降到 $O(h^3)$**——每步多一次 NFE，换来一个数量级的精度提升。

### RF-Edit：基于 feature sharing 的编辑框架

建立在 RF-Solver 的精确 inversion 之上：

1. **Inversion 阶段**：用 RF-Solver 反推噪声，同时**存储**最后 $M$ 个 transformer block 在最后 $n$ 步的 self-attention **Value features** $\{\hat{V}_{t,k}^m\}$
2. **Editing 阶段**：用新 prompt 做 denoise，在对应位置**注入**存储的 Value features 替换原生 Value：

$$F'_{t,k}^m = \text{Attention}(Q_{t,k}^m, K_{t,k}^m, \tilde{V}_{t,k}^m)$$

这样 Query/Key 跟随新 prompt 产生编辑语义，而 Value 保留源图结构信息——实现"改内容不改结构"。

**架构适配**：
- **FLUX（图像）**：在 Single Blocks（统一处理 text-image features）中注入
- **OpenSora（视频）**：在 spatial attention 模块中注入，保持帧间结构一致

### 关键优势

- 完全 **training-free**：不需要额外训练或优化
- 兼容任何预训练 rectified-flow 模型
- 每步额外开销仅 +1 NFE（有限差分）

## Results

### Inversion / Reconstruction

| 指标 | Vanilla RF | RF-Solver | 改善 |
|------|-----------|-----------|------|
| Image MSE | 0.0268 | **0.0092** | -66% |
| Image LPIPS | 0.6253 | **0.4239** | -32% |
| Video MSE | — | — | -35% |

### Generation（采样质量）

| 指标 | Vanilla RF | RF-Solver |
|------|-----------|-----------|
| FID | 25.33 | **23.98** |
| CLIP Score | 31.01 | **31.09** |

RF-Solver 在正向生成上也有提升（更精确的 ODE 积分 → 更好的样本质量）。

### Image Editing

| 方法 | CLIP Score | LPIPS |
|------|-----------|-------|
| RF-Inversion | 33.02 | — |
| **RF-Edit** | **33.66** | 0.149 |

### Video Editing

| 方法 | Semantic Consistency | Motion Smoothness |
|------|---------------------|-------------------|
| TokenFlow | 0.9439 | — |
| **RF-Edit** | **0.9501** | **0.9712** |

## 关系

### 与已有 wiki 页的关联

- **[[wiki/methods/rectified-flow|Rectified Flow]]**：RF-Solver 专门针对 RF 的 ODE 形式推导，利用 RF 路径接近直线的特性（使 Taylor 展开快速收敛）
- **[[wiki/concepts/flow-matching|Flow Matching]]**：RF 是 FM 的特例（OT coupling），RF-Solver 的 Taylor 展开思路对一般 FM 模型也适用
- **[[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS Solver]]**：BNS 离线学 solver 系数（model-specific, not instance-aware）；RF-Solver training-free 且利用局部高阶信息——两者路线不同但都改善 ODE 积分精度
- **[[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solver 2023]]**：同属"改进 solver 而非改模型"路线
- **[[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]]**：FDS 做 spatial refinement（移动 $x_t$），RF-Solver 做 temporal refinement（更精确积分）——正交
- **[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]]**：HRF 通过改训练解决速度平均化；RF-Solver 不处理速度平均化，只处理离散化误差——解决不同层面的问题

### 对我的 thesis 的启示

**判断：中度相关。RF-Solver 是 temporal 精度路线的代表，与你的方向在技术层面有交集但解决不同问题。**

关键区分：
1. **RF-Solver 假设速度场本身是准确的**——它处理的是"$v_\theta$ 对了但步太大"的问题。你关注的是"$v_\theta$ 在交叉区域本身就错了（被平均化）"——即使 RF-Solver 完美复现 ODE 轨迹，如果轨迹本身经过速度被平均化的区域，结果仍然是 OOD 的
2. **Taylor 展开在速度场光滑区收敛快，在奇异区可能失效**：如果 $v_\theta$ 在某个 $t$ 处有剧烈变化（正是奇异区的特征），高阶展开反而可能放大误差。这是 RF-Solver 可能在你关心的 case 上表现不佳的理论原因
3. **Feature sharing 思路与 editing 相关**：RF-Edit 的 Value 注入策略对你的 editing pipeline 有参考价值——但它处理的是"保结构"问题，不是"避 OOD"问题

**可借鉴**：
- 有限差分估计导数的技巧简单有效，可以用于检测速度变化剧烈程度（作为奇异区检测的信号之一）
- RF-Edit 的 feature sharing 可以和你的 adaptive solver 组合：先用 adaptive 策略安全通过奇异区，再用 feature sharing 保持结构一致性

## Open Questions

- [ ] RF-Solver 的 Taylor 展开在速度场不光滑（奇异区）处的行为？是否有数值不稳定的实验证据？
- [ ] RF-Solver + adaptive step-size：在检测到 $v_\theta^{(1)}$ 异常大时自动缩小/增大步长？
- [ ] RF-Edit 的 Value sharing 能否与 FDS 的 divergence signal 结合——高 divergence 区域用更多 Value 注入来稳定？
- [ ] 有限差分的 $\Delta t = 0.01$ 是否对所有模型都适用？FLUX 级别模型的最优 $\Delta t$？
- [ ] RF-Solver 在 video editing 中 temporal consistency 是否依赖于 spatial attention sharing，还是 solver 精度本身就足够？

## 出处

- arXiv: [2411.04746](https://arxiv.org/abs/2411.04746)
- 代码: https://github.com/wangjiangshan0725/RF-Solver-Edit
- 作者: Jiangshan Wang, Junfu Pu, Zhongang Qi, Jiayi Guo, Yue Ma, Nisha Huang, Yuxin Chen, Xiu Li, Ying Shan（清华 / 腾讯 ARC Lab）
