---
type: source
title: "What Does Guidance Do? A Fine-Grained Analysis in a Simple Setting"
aliases: [What Does Guidance Do, "Chidambaram et al. 2024"]
tags: [diffusion, guidance, classifier-free-guidance, probability-flow-ode, score-error]
status: active
created: 2026-08-14
updated: 2026-08-14
raw: "[[raw/literature-notes/chidambaramWhatDoesGuidance2024]]"
authors: "Muthu Chidambaram, Khashayar Gatmiry, Sitan Chen, Holden Lee, Jianfeng Lu"
venue: NeurIPS 2024
year: 2024
arxiv: "2409.13074"
---

# What Does Guidance Do? A Fine-Grained Analysis in a Simple Setting

> Guidance 不是对终点分布做一次静态重加权，而是改变 reverse dynamics 的时变 transport：它把样本推向目标类中远离竞争类别的极端区域，并在过强时放大低密度区域的 score error。

## Motivation

Guidance 常被解释为从
$$p^{z,w}(x)\propto p(x)p(z\mid x)^{1+w}$$
采样，因为 $t=0$ 时
$$\nabla\log p^{z,w}=(1+w)\nabla\log p(x\mid z)-w\nabla\log p(x).$$

但 diffusion / [[wiki/concepts/probability-flow-ode|PF-ODE]] 在每个噪声时刻都需要 score。目标 tilted distribution 所需的是“先 tilt、再加噪”的 $\nabla\log p_t^{z,w}$；实际 guidance 使用“先加噪、再逐时 tilt”的
$$(1+w)\nabla\log p_t(x\mid z)-w\nabla\log p_t(x).$$
Noising 与 tilting 一般不交换，所以逐时 guided score 的形式成立，不代表终点服从朴素 tilted distribution。

## Method

作者在一维二分类的可解析设置中研究 guided PF-ODE：两个互不相交的紧支撑分布，以及 $\tfrac12\mathcal N(1,1)+\tfrac12\mathcal N(-1,1)$。

核心直觉是：状态越像错误类，guidance correction 越强；已经明显属于目标类时，额外 correction 越弱。因此靠近竞争类的一侧受到更强推力，轨迹偏向目标类内部离竞争类最远的位置。紧支撑情形还出现“强推 → 越过支撑 → 进入尾部 → 回拉 → 停在边界附近”的 overshoot–return。

- **Theorem 1（正式版 Theorem 4）**：目标类支撑为 $[\alpha_1,\alpha_2]$ 时，充分大的 $w$ 使样本以 $1-e^{-\Omega(w)}$ 的概率集中到距 $\alpha_2$ 为 $O(1/\sqrt{\log w})$ 的边界层。
- **Theorem 2**：高斯混合正类引导满足
  $$\Pr[\tilde x(1)\ge0]\ge1-e^{-\Omega(w^2)},\quad
  \Pr[\tilde x(1)\ge\sqrt w]\ge1-e^{-\Omega(w)},$$
  即无有限边界时，同一机制表现为深入目标分布尾部。
- **Theorem 3**：即使整体 $L^2$ score error 非零但很小，充分大的 guidance 仍可使样本以至少 $1-e^{-\Omega(w)}$ 的概率 off-support。$L^2(p_t)$ 对低密度 tail 的误差约束弱，而 guidance 主动把轨迹送入这些区域。

## Results

实验主要是机制的定性验证：

- synthetic mixture 复现 boundary concentration、overshoot 与 pullback；
- MNIST 采用 one-vs-all 投影、每类先生成 100 个样本估计均值方向，采样使用 400-step DDPM；digit 0 及多数其他 digit 上，定性最佳的 $w=3$ 也是仍保持投影轨迹单调的最大候选值；
- ImageNet-256 每侧生成 50 个样本估计投影方向，采样使用 25-step DDIM。该数据不满足一维理论假设：即使 $s=25$ 也不复现 MNIST 的非单调轨迹；tiger 类中 $s=5$ 是尚无 RGB support error 的最大候选值，更大 scale 出现视觉异常。其他类别的最佳区间不同，basketball 等高方差类别甚至不满足 support error 随 scale 单调增加。

因此实践 heuristic 只能表述为：**在最终样本仍位于可信数据区域的前提下，把 guidance 设得尽可能大**。Sweet spot 可能对应尚未明显进入 overshoot / off-support regime 时的最大 scale；ImageNet 结果只是最小定性演示，不是通用阈值。

## 关系

- [[wiki/concepts/classifier-free-guidance]] / [[wiki/concepts/classifier-guidance]]：理论同时适用于两种实现；关键是逐时外推，不是条件信号来自哪个网络。
- [[wiki/concepts/probability-flow-ode]]：论文分析 guided PF-ODE；引导后不再保持原条件分布的边缘路径。
- [[wiki/sources/dhariwalDiffusionModelsBeat2021|Classifier Guidance]] 与 [[wiki/sources/hoClassifierFreeDiffusionGuidance2022|CFG 原文]]建立质量—多样性权衡；本文解释原型化、尾部采样和质量回落。
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]]说明收益依赖 noise interval；本文从动力学侧说明 guidance 不能只由终点静态 density 解释。
- 本文不证明 [[wiki/concepts/flow-matching]] / [[wiki/methods/rectified-flow]] 的尖锐转折来自相同机制，也不验证 Reject-and-Skip 的恢复区假设。

## 对我的 thesis 的启示

1. **不改变 working thesis 版本。** 真实 FM 中的曲率事件与恢复区仍需独立验证。
2. Guidance scale 还改变轨迹访问的状态分布；进入低密度区后，score / velocity error 与数值误差都可能更危险。
3. 可把 conditional–unconditional disagreement、非单调性或 off-support proxy 作为“何时减弱/关闭 guidance”的候选信号，但目前仍是假设。
4. 简单分布存在 pullback，不代表真实高维模型跨过危险区后仍能恢复；这强化了出口验证的必要性。

## 待调研方向

- [ ] 如何在高维图像空间定义 trajectory monotonicity / off-support proxy？
- [ ] 能否联合 guidance interval 与在线 score-reliability proxy，自适应选择 $w(t,x_t)$？
- [ ] 结论如何迁移到 Flow Matching / Rectified Flow guidance？

## 出处

- [[raw/literature-notes/chidambaramWhatDoesGuidance2024]]
- NeurIPS 2024 Main Conference；arXiv: 2409.13074
