---
type: concept
title: Reject-and-Skip
aliases: [reject-and-skip flow solver, rollback-and-skip, 拒绝并跨越]
tags: [flow-matching, solver, transport-coupling, adaptive-inference]
status: active
created: 2026-08-03
updated: 2026-08-14
sources: ["[[wiki/sources/jainDiffusionTreeSampling2025]]"]
evidence: ["[[research/experiments/2026-08-02-reject-and-skip-toy-report]]", "[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]"]
---

# Reject-and-Skip

## 一句话定义

一种面向 Flow Matching 局部不可信速度场的推理策略：trial step 若触发异常检测，则回滚到上一个可信状态，不只缩小步长，而是搜索异常区之后的可信出口并跨越，从而有意改变离散 transport coupling。

## 动机

Conditional Flow Matching 的最优边缘速度是条件速度的后验平均：

$$
v(x,t)=\mathbb E[u_t\mid X_t=x].
$$

当多个传输分支局部接近、交叉或后验混合时，这个平均场在分布层面可能正确，却可能让有限步数的单条轨迹减速、转向、进入分支间低密度区或切换 coupling。传统 adaptive solver 的目标是更精确地追踪该 marginal ODE；reject-and-skip 的假设是某些局部区域不值得忠实解析。

## 基本流程

```text
accepted state
  → ordinary trial
  → detect
      ├─ trustworthy: accept/correct
      └─ suspicious: rollback
           → scan farther exits
           → validate first recovered exit
               ├─ valid: accept/correct skip
               └─ none valid: shrink/fallback
```

对入口 $(x_0,t_0)$，复用 $v_0=v(x_0,t_0)$，构造

$$
x_{\mathrm{trial}}^{(m)}=x_0+mhv_0,
\qquad
v_m=v(x_{\mathrm{trial}}^{(m)},t_0+mh),
$$

并扫描

$$
D(m)=\frac{\lVert v_m-v_0\rVert}{\tfrac12(\lVert v_m\rVert+\lVert v_0\rVert)+\epsilon}
$$

或速度夹角。研究中的候选 return pattern 是 $D(m)$ 先升高、后下降，表示局部速度异常之后可能重新接近入口方向。$D(1)$ 可复用 Heun trial 的两次 velocity evaluation；每个额外候选出口通常只增加 1 NFE。

## 与传统 adaptive solver 的区别

| 维度 | Adaptive shrink / step-doubling | Reject-and-skip |
|---|---|---|
| 目标 | 减小相对细步 ODE 的截断误差 | 跨过局部不可信场并寻找可信出口 |
| 困难时动作 | 缩小步长、增加局部分辨率 | 回滚、扩大候选跨度，失败才 shrink |
| 主要判据 | embedded defect / step-doubling | 状态可信度、velocity return、出口验证 |
| 对 coupling | 尽量保留 marginal ODE coupling | 允许主动改变离散 coupling |
| 最终评价 | reference error | 同 NFE 生成分布与尾部质量 |

两者并非完全互斥：adaptive Heun 在大步跨区时也可能偶然改变 coupling。因此 rollback、非 oracle detector 和出口验证是否提供额外稳定收益，是本方向的核心对照问题。

## 当前证据边界

- **Toy observation**：oracle ambiguity detector 下，约 15–17 NFE 的 skip + corrector 降低了粗 Euler 的 endpoint outlier，并以很大的 RK4 RMSE 保留约 98%–99% 人工分支。
- **Toy conclusion**：该机制实施的是结构化 coupling change，而非更准确的 marginal ODE integration。
- **Real-model observation**：CIFAR-10 1-RF 存在可检测的数值困难与强 endpoint boundary layer；velocity-based 信号能预测局部截断误差。
- **Unknown**：内部困难是否对应 conditional ambiguity；更远候选是否出现稳定 velocity return；skip 是否在严格同 NFE 下改善生成分布。

## 与 Diffusion Tree Sampling 的新颖性边界

[[wiki/methods/diffusion-tree-sampling|DTS]] 已占据“生成轨迹分支搜索 + terminal reward backup + 跨 rollout 复用”这一宽泛位置。两者当前仍有明确边界：

| 维度 | DTS | Reject-and-Skip |
|---|---|---|
| 底座 | stochastic reverse Markov chain | deterministic Flow Matching ODE |
| 范围 | 跨 rollout 的全局持久树 | 单轨迹、局部触发 |
| 信号 | clean endpoint reward / soft value | 在线状态或 velocity 可信度 |
| 预算 | 高 branching NFE、anytime scaling | 严格等低 NFE |
| 动作 | selection / expansion / rollout / backup | trial / reject / rollback / skip |

因此，branch search、lookahead 或 terminal feedback 本身不能作为本方向的新意；若未来引入这些组件，必须以 DTS 为直接 prior art。当前可守的研究命题是：**不依赖 terminal reward 和持久树，能否用低成本 state detector 在 deterministic flow 中完成局部 coupling intervention。**

## 成功判据

1. 非 oracle detector 在独立校准种子上稳定触发，而非只编码全局时间位置。
2. 出口验证与状态可信度或生成失败相关，而非只测 ODE truncation error。
3. 严格记录逐样本 NFE，并与 fixed midpoint/RK4、adaptive Heun、shrink-only 和无 rollback 大步对照。
4. 在多随机种子下改善 FID/KID、precision/recall、feature OOD 或重尾失败；reference RMSE 仅作 coupling-change diagnostic。

## 与其他概念的关系

- 理论来源：[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/flow-matching]]
- 解释核心：[[wiki/concepts/transport-coupling]]
- 方法定位：[[wiki/concepts/ode-solver-taxonomy]] 中的 instance-aware temporal intervention
- 搜索型直接 prior art：[[wiki/methods/diffusion-tree-sampling|DTS]]
- Backbone：[[wiki/methods/rectified-flow]]
- 研究路线：[[wiki/synthesis/reject-and-skip-research-direction]]

## 实验证据

- [[research/experiments/2026-08-02-reject-and-skip-toy-report]]
- [[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]
