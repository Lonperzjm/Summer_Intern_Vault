---
type: concept
title: ODE Solver Taxonomy（FM/Diffusion 采样求解器谱系）
aliases: [ODE solver taxonomy, solver taxonomy, 求解器谱系]
tags: [flow-matching, diffusion, solver, ode, sampling]
status: active
created: 2026-07-31
updated: 2026-08-04
sources: ["[[wiki/sources/shaulBespokeSolversGenerative2023]]", "[[wiki/sources/shaulBespokeNonStationarySolvers2024]]", "[[wiki/sources/wangTamingRectifiedFlow2025]]", "[[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026]]", "[[wiki/sources/chaTrainingFreeRefinementFlow2026]]", "[[wiki/sources/chenBiAnchorInterpolationSolver2026]]", "[[wiki/sources/flow-matching-solver-method-taxonomy]]"]
evidence: ["[[research/experiments/2026-08-02-reject-and-skip-toy-report]]", "[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]"]
---

# ODE Solver Taxonomy（FM/Diffusion 采样求解器谱系）

> 概念页。综合 [[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS]] 的 solver taxonomy 定理、vault 已 ingest 的 solver 论文与[[wiki/sources/flow-matching-solver-method-taxonomy|用户整理的 solver 分类]]，梳理 FM/diffusion 模型采样加速的求解器全景。

## 核心前提

FM/diffusion 模型的确定性采样归结为求解 ODE：

$$\frac{dx_t}{dt} = v_\theta(x_t, t)$$

采样质量和速度都取决于**如何离散化这个 ODE**。所有 solver 改进都在"$v_\theta$ 本身正确"的假设下工作——如果速度场在某些区域被多模态平均化（参见 [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]] / [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]] 的分析），solver 改进无法修复这个问题。

## 传统数值基线与比较口径

“步数”不能代替成本口径。至少应区分：时间步数、backbone NFE、理论全局阶数、自适应接受/拒绝、历史缓存，以及辅助网络、Jacobian/VJP、通信和编译成本。

| 家族 | 代表 | 主要价值 | 主要代价或边界 |
|---|---|---|---|
| 显式单步 | Euler、Midpoint、Heun、RK3、RK4 | 固定步、无历史；适合可控基线 | 高阶通常需要更多当步 NFE；困难区可能降阶 |
| 嵌入式 RK | RK45、Tsit5 | 局部误差控制；参考轨迹与诊断 | 动态 NFE、拒步；批处理与固定延迟不友好 |
| 线性多步 / 预测—校正 | Adams–Bashforth、ABM | 热启动后用历史换低新 NFE | 有热启动、历史和稳定域问题；变步长系数复杂 |
| 隐式刚性法 | BDF、Radau | 刚性诊断与小模型参考 | 非线性迭代/Jacobian 对大型网络通常过贵 |
| 随机积分 | Euler–Maruyama、stochastic Heun | 反向 SDE 或随机流基线 | 不能直接替代纯确定性 FM ODE solver |

Euler 每步 1 NFE，局部截断误差 $O(h^2)$、全局误差 $O(h)$。但“stage 数 = 方法阶数”不是一般规律；阶数依赖 Runge–Kutta 阶条件、速度场光滑性和稳定性假设。详见 [[wiki/sources/flow-matching-solver-method-taxonomy]]。

## BNS 的包含链（Theorem）

[[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS（Shaul et al. 2024）]] 证明了严格的 solver 族包含关系：

```
Euler ⊂ RK (Runge-Kutta)
         ⊂ Exponential-RK
              ⊂ Scale-Time RK [Bespoke 2023]
                   ⊂ Non-Stationary Solvers [BNS 2024]

Adams-Bashforth ⊂ Multistep
                    ⊂ Exponential-Multistep
                         ⊂ Scale-Time Multistep
                              ⊂ Non-Stationary Solvers
```

含义：DDIM、DPM-Solver、指数积分器、带 scheduler/time reparameterization 的方法（包括 [[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke 2023 的 scale-time 族]]）都是 Non-Stationary solver 参数空间中的特殊情况。

## 分类维度

### 按是否需要额外训练

| 类别 | 代表 | 参数量 | 训练成本 |
|------|------|--------|---------|
| **Training-free** | Euler, Heun, RK4, [[wiki/sources/wangTamingRectifiedFlow2025\|RF-Solver]], [[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026\|FastFlow]], CAB | 0 | 0 |
| **Solver distillation**（离线优化少量 solver 参数） | [[wiki/sources/shaulBespokeSolversGenerative2023\|Bespoke]], [[wiki/sources/shaulBespokeNonStationarySolvers2024\|BNS]], BOSS, Differentiable Solver Search | <200 | ~1% 原模型训练 |
| **Learnt velocity interpolator**（训练轻量 sideband 网络） | [[wiki/sources/chenBiAnchorInterpolationSolver2026\|BA-solver]] | ~6M | ~250 iter（极低，但高于 solver distillation） |
| **模型蒸馏**（改模型权重） | Progressive Distillation, Consistency Model | 全模型 | 高 |

### 按改进维度

| 维度 | 含义 | 代表 |
|------|------|------|
| **Temporal 精度** | 提高每步积分精度 | [[wiki/sources/wangTamingRectifiedFlow2025\|RF-Solver]]（Taylor 展开）、DPM-Solver（指数积分）、[[wiki/sources/chenBiAnchorInterpolationSolver2026\|BA-solver]]（SideNet 区间内插值 + 高阶求积） |
| **Temporal 分配** | 优化步长/时间网格 | [[wiki/sources/shaulBespokeSolversGenerative2023\|Bespoke]]（scale-time 变换）、BOSS（动态规划）、SharpEuler |
| **Temporal 跳步** | 自适应省略部分步 | [[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026\|FastFlow]]（bandit 选跳步）|
| **Temporal coupling 干预** | 检测局部不可信场后回滚并跨区，允许偏离 marginal ODE | [[wiki/concepts/reject-and-skip]]（研究中） |
| **Temporal 全历史** | 利用所有历史速度评估 | [[wiki/sources/shaulBespokeNonStationarySolvers2024\|BNS]]（非平稳多步法）、CAB（修正 Adams-Bashforth） |
| **Spatial 修正** | 在 state space 挪位置 | [[wiki/sources/chaTrainingFreeRefinementFlow2026\|FDS]]（divergence 引导）|

Temporal 和 Spatial 维度正交，可组合（如 BNS + FDS）。

### 按改动对象：避免把所有加速都叫 solver

除传统积分公式外，还应分开记录：模型专用参数优化、轻量辅助网络、training-free 结构化积分、时间网格/预算、轨迹选择、扩散参数化适配，以及系统/并行加速。它们通常可以跨层组合，评价时应固定其余层并逐层消融。

特别是 FDS 与 [[wiki/concepts/reject-and-skip]] 更接近 sampler / coupling intervention；SADA、ParaFlow 更接近系统加速。前者不以最小化原 ODE 离散误差为唯一目标，后者也不一定减少数学意义上的 NFE。

### 按是否 instance-aware

| 类型 | 代表 | 说明 |
|------|------|------|
| **全局固定**（per-model） | Bespoke, BNS, BOSS | 所有样本共享同一 solver；离线优化 |
| **Dataset-level adaptive** | [[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026\|FastFlow]] | bandit 跨样本在线学习，但不以当前 $x_t$ 为 context |
| **Instance-aware** | FDS, adaptive RK45, [[wiki/sources/chenBiAnchorInterpolationSolver2026\|BA-solver]], [[wiki/concepts/reject-and-skip]] | 每个样本独立决策；基于当前状态 |

## 与速度平均化 / 奇异区问题的关系

**所有 solver 改进都假设 $v_\theta$ 本身正确**。但在多模态交叉区域，$v_\theta = \mathbb{E}[v \mid x_t]$ 本身是多个模态的平均——此时即使 solver 完美复现 ODE 轨迹，轨迹本身就指向 OOD 区域。

这是 solver 改进的盲区，也是"奇异区自适应干预"方向的 novelty 空间：

| 问题层面 | Solver 改进能解决吗？ | 需要什么？ |
|---------|---------------------|-----------|
| 离散化误差（步太大） | ✅ BNS/RF-Solver/高阶法 | 更好的离散化 |
| 速度场 bias（被平均化） | ❌ | spatial correction（FDS）或 training-based（HRF）或 per-sample adaptive 干预 |
| 奇异区 OOD | ❌ | 事前检测 + 干预策略（跨步/绕行） |

[[wiki/concepts/reject-and-skip]] 位于 taxonomy 的边界：形式上它是 temporal skip，目标上却不是提高 ODE integration accuracy，而是通过 rollback、远端出口搜索和状态验证主动改变离散 coupling。因此其主指标不能沿用 solver reference RMSE，必须使用严格等 NFE 的分布质量与失败尾部；reference error 只记录它偏离 marginal ODE 的程度。

## 关系

- [[wiki/concepts/flow-matching]]：所有 solver 都在求解 FM 的 ODE
- [[wiki/concepts/probability-flow-ode]]：PF-ODE 是 score-based 侧的对应物
- [[wiki/methods/rectified-flow]]：reflow 与 solver 正交——前者改轨迹形状，后者改离散化
- [[wiki/concepts/reflow]]：reflow 后轨迹接近直线→ solver 需求降低，外推更准
- [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]] / [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]]：处理 solver 无法解决的速度平均化问题
- [[wiki/concepts/reject-and-skip]]：研究中的 instance-aware rollback-and-skip，尝试从时间维主动跨过局部不可信场

## 出处

- BNS solver taxonomy: [[wiki/sources/shaulBespokeNonStationarySolvers2024]]
- 传统数值法、专用方法分层与成本口径: [[wiki/sources/flow-matching-solver-method-taxonomy]]
- 原始全景表: `raw/notes/flowmatching_solver_methods_2026-07-29.xlsx`
