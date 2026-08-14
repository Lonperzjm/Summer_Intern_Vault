---
type: method
title: "Diffusion Tree Sampling"
aliases: [DTS, Diffusion Tree Search, "DTS*"]
tags: [diffusion, guidance, inference-time-alignment, tree-search, mcts]
status: active
created: 2026-08-14
updated: 2026-08-14
sources: ["[[wiki/sources/jainDiffusionTreeSampling2025]]"]
family: guidance
---

# Diffusion Tree Sampling

## 一句话总结

把 stochastic reverse diffusion 展开成持久化搜索树，用 terminal reward 的 soft Bellman backup 累积中间状态 value，使后续 rollout 复用此前推理计算。

## 目标分布

$$
\pi^*(x_0)\propto p_\theta(x_0)e^{\lambda r(x_0)},\qquad
V_t(x_t)=\lambda^{-1}\log\mathbb E[e^{\lambda r(x_0)}\mid x_t].
$$

理想 transition 满足

$$
\pi_t^*(x_{t-1}\mid x_t)\propto
p_\theta(x_{t-1}\mid x_t)e^{\lambda V_{t-1}(x_{t-1})}.
$$

DTS 的核心就是用实际 rollout 与树上 backup 逐步估计这里的 $V$。

## Tree iteration

1. **Selection**：沿已建树选择 child；
2. **Expansion**：从 base reverse transition 采新 child，progressive widening 限制分支；
3. **Rollout**：完成到 clean endpoint 的去噪，并保存整条路径；
4. **Backup**：评估 terminal reward，向 ancestors 做 log-sum-exp soft backup。

## 两种输出语义

| 变体 | Selection | 目标 |
|---|---|---|
| DTS | Boltzmann over soft value | reward-tilted posterior sampling |
| DTS$^\star$ | soft-value greedy + UCT exploration | 高 reward global search / marginal-MAP |

DTS 在 bounded reward 与无限 iterations 下有渐近一致性；DTS$^\star$ 不应被描述为 exact posterior sampler。

## 成本与边界

- 冻结 backbone，不训练 reward-specific value network；reward 只需在 endpoint 可评。
- 树构建需要大量 serial rollouts、状态缓存与 branching NFE；按 timestep batching 只能部分并行。
- 树建成后可近零 NFE 重复采样已有 leaves。
- 方法的自然 branching 来自 stochastic reverse transition；deterministic ODE / Flow Matching 的分支机制未解决。

## 关系

- [[wiki/concepts/training-free-guidance]]：用 rollout aggregation 替代单点 $r(\hat x_0)$ 的高噪声 value approximation。
- [[wiki/concepts/tweedie-formula]]：DTS 的直接诊断对象。
- [[wiki/concepts/reject-and-skip]]：同属 trajectory selection，但 DTS 是跨 rollout 的 global reward search，Reject-and-Skip 是 deterministic flow 上的局部 state-triggered intervention。
- Best-of-$N$：不复用轨迹；SMC：复用 particle population，但不持久化完整 denoising tree 做全局 backup。

## 出处

- [[wiki/sources/jainDiffusionTreeSampling2025]]
