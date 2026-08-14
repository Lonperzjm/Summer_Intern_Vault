---
type: source
title: "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"
aliases: ["Jain et al. 2025", Diffusion Tree Sampling, DTS]
tags: [diffusion, guidance, inference-time-alignment, tree-search, mcts, reward]
status: active
created: 2026-08-14
updated: 2026-08-14
raw: "[[raw/literature-notes/jainDiffusionTreeSampling2025]]"
authors: "Vineet Jain, Kusha Sareen, Mohammad Pedramfar, Siamak Ravanbakhsh"
venue: arXiv preprint
year: 2025
arxiv: "2506.20701"
---

# Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models

> Jain et al., arXiv:2506.20701v1, 2025。把 inference-time alignment 建模成可跨 rollout 复用信息的 denoising tree，以 terminal reward 的 soft backup 改善高噪声 value estimation。

## Motivation

对冻结生成模型 $p_\theta$ 和只在 clean endpoint 可评的 reward $r$，目标是

$$
\pi^*(x_0)=Z^{-1}p_\theta(x_0)\exp\{\lambda r(x_0)\}.
$$

相应 soft value 为

$$
V_t(x_t)=\frac1\lambda\log\mathbb E_{p_\theta(x_{0:t-1}\mid x_t)}
\left[\exp\{\lambda r(x_0)\}\right].
$$

[[wiki/concepts/training-free-guidance|Training-Free Guidance]]、SMC 和局部 search 常以 $r(\hat x_0(x_t))$ 或单次 rollout 近似它。论文指出：[[wiki/concepts/tweedie-formula|Tweedie clean estimate]] 在高噪声阶段可能近似随机；独立 Best-of-$N$ / particle run 还会丢掉此前轨迹积累的 endpoint 信息。

## Method

[[wiki/methods/diffusion-tree-sampling|DTS]] 把 reverse Markov chain 变成树：节点保存 $(x_t,t,\hat v_t,N_t)$，边由随机 reverse transition $p_\theta(x_{t-1}\mid x_t)$ 产生。每轮包含：

1. **Selection**：DTS 按 $\exp(\lambda\hat v)$ 的 Boltzmann policy 选已有 child；
2. **Expansion**：用 base diffusion transition 产生新 child，并以 progressive widening 控制分支数；
3. **Rollout**：从新节点运行到 $x_0$，且把整条 rollout 加入树；
4. **Backup**：令 $\hat v_0=r(x_0)$，用 soft Bellman backup 更新所有 ancestors。

这把 additional compute 从“更多独立样本”变成“更准确的共享 value landscape”。树建成后，从树中采样不再调用生成模型。

### DTS 与 DTS$^\star$

- **DTS**：Boltzmann selection，目标是从 $\pi^*$ 采样。论文 Proposition 1 在 bounded reward、$\lambda>0$、无限 tree iterations 下给出 empirical distribution 的渐近一致性。
- **DTS$^\star$**：用 soft-value greedy selection，并加入 UCT exploration bonus，目标是 dominant posterior region 内的高 reward sample，属于 marginal-MAP / global search，不宣称保持 DTS 的 posterior-sampling 语义。

## Results

- 在 2D mixture/checkerboard 上，DTS 随 NFE 增加持续降低对目标分布的 MMD；tree aggregation 相比 Tweedie one-step 和 single rollout 降低 value estimate 的 bias/variance。
- $10^6$ NFE 的 CIFAR-10 class-conditional 实验中，DTS FID **0.195**，对比 DAS 0.241、SMC/FK 0.313、DPS 0.486。
- 摘要报告：MNIST/CIFAR-10 可用最高约 **10× 更少 compute** 匹配最强 baseline；text-to-image 与 language completion 相对 Best-of-$N$ 最高约 **5×**。
- SD v1.5 文生图中，DTS$^\star$ 随 compute scaling 优于 Best-of-$N$；SMC 的 reward 有时更高但出现可见 over-optimization。MDLM 文本实验中 DTS$^\star$ 同时保持较高 reward 与 trigram diversity。

## 结果边界

- “渐近精确”只属于 DTS sampling variant，并依赖 bounded reward、正 temperature 与无限 rollout；不能外推为有限预算下无偏。
- 高维树搜索仍靠 pretrained model 作为强 prior，否则分支空间指数增长。
- Tree control flow 比 SMC 更串行；作者虽按 timestep batching，但大规模并行效率和 wall-clock 不等同于 NFE。
- 论文实验使用 stochastic reverse diffusion / discrete diffusion。Deterministic probability-flow ODE 或 [[wiki/concepts/flow-matching|Flow Matching]] 没有天然随机 child，文中没有给出对应 tree construction。
- text-to-image 的“不过度优化”主要由定性样本与 soft-mass 解释支持，不是 reward hacking 的普适保证。

## 关系

- **方法页**：[[wiki/methods/diffusion-tree-sampling]]。
- **高噪声诊断**：为 [[wiki/concepts/tweedie-formula]] 与 clean-estimate guidance 的 Jensen / posterior-width caveat 提供直接 bias–variance 实验。
- **基线关系**：Best-of-$N$ 独立采样；SMC 在同一 denoising run 内维护 particles；DTS 跨 rollout 保存树并回传 terminal reward。
- **当前研究边界**：[[wiki/concepts/reject-and-skip|Reject-and-Skip]] 是 deterministic FM 上的低预算、state-triggered 单轨迹局部干预；DTS 是 stochastic chain 上的高预算、reward-driven 全局树搜索。若前者引入多分支 lookahead 或 terminal reward backup，必须把 DTS 作为直接 prior art。

## 对我的 thesis 的启示

1. **不改变 working thesis 版本。** 论文没有 Flow Matching、deterministic ODE 或 editing 实验。
2. **收窄新颖性边界。** “轨迹分支搜索”“终点反馈回传”“复用过去 rollout”本身均不能再作为 Reject-and-Skip 的新意；当前可守边界是严格等 NFE、单轨迹在线触发、局部 rollback/skip 与无需 terminal reward。
3. **提供强对照轴。** 如果未来允许更高推理预算，应区分 independent Best-of-$N$、particle SMC、DTS global tree 与 Reject-and-Skip local intervention，分别报告 NFE、wall-clock、串行深度和缓存。

## 待调研方向

- [ ] 给 deterministic ODE 注入何种受控 perturbation 才能形成有意义分支，同时不把问题退化成 stochastic sampler？
- [ ] 能否只在 detector 触发的少数节点做 shallow tree rollout，连接 DTS 的 value backup 与 Reject-and-Skip 的低预算局部干预？
- [ ] DTS 的缓存、节点数与显存随 rollout budget 如何扩展？
- [ ] 在 editing 中，terminal reward 如何同时表达 instruction adherence 与 source fidelity，避免单一 reward 过优化？

## 出处

- [[raw/literature-notes/jainDiffusionTreeSampling2025]]
- arXiv: 2506.20701v1
