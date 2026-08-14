---
type: source
title: "Learn to Guide Your Diffusion Model"
aliases: ["Galashov et al. 2025", Learn to Guide]
tags: [diffusion, flow-matching, guidance, classifier-free-guidance, self-consistency]
status: active
created: 2026-08-14
updated: 2026-08-14
raw: "[[raw/literature-notes/galashovLearnGuideYour2025]]"
authors: "Alexandre Galashov, Ashwini Pokle, Arnaud Doucet, Arthur Gretton, Mauricio Delbracio, Valentin De Bortoli"
venue: arXiv preprint
year: 2025
arxiv: "2510.00815"
---

# Learn to Guide Your Diffusion Model

> Galashov et al., arXiv:2510.00815v1, 2025。冻结原生成模型，学习依赖 condition 与两端时间的 CFG 权重，而不是继续手调全局 scale。

## Motivation

固定 [[wiki/concepts/classifier-free-guidance|CFG]] 默认不同条件与阶段需要相同修正。作者改把 CFG 理解为对 imperfect conditional denoiser 的 correction：误差随 condition/time 改变，最优权重也不应固定。

## Method

学习
$$
\omega^\phi_{c,(s,t)}=\omega(s,t,c;\phi),\qquad 0\le s<t\le1,
$$
其中 $t$ 是 proposal time，$s$ 是目标 time。conditional/unconditional denoiser 或 velocity model 均冻结，只训练带非负输出的小型 guidance network。

监督来自 **self-consistency**：对真实 $(x_0,c)$，匹配两条路径在 $s$ 的分布：

1. $x_0\rightarrow x_s$：直接 forward noising；
2. $x_0\rightarrow x_t\rightarrow\tilde x_s$：先加噪到 $t$，再以 learned guidance 回到 $s$。

主目标用 energy-MMD；generated–target 项负责靠近目标，generated–generated 排斥项抑制坍塌。$\ell_2$ 简化版相当于 $\beta=2,\lambda=0$，省去 $O(m^2)$ 粒子比较，但更敏感。推理相邻步差很小，训练用约 $\delta\approx0.1$ 的较大 gap 反而更好。扩展版加入 clean-data reward，并用 consistency regularization 约束 reward optimization。

## Results

- ImageNet 64：unguided / constant CFG / Limited Interval / learned guidance 的 FID 为 **4.46 / 2.40 / 2.11 / 1.99**。
- CelebA：unguided 2.44、Limited Interval 2.37、learned guidance **2.10**。
- MS-COCO 512：冻结 1.05B [[wiki/concepts/flow-matching|Flow Matching]] 模型时，unguided 为 FID 24.74 / CLIP 0.278，learned guidance 为 **18.01 / 0.295**；加入 CLIP reward 后为 28.37 / **0.306**，显示 alignment 与 distributional fidelity 仍有权衡。
- 学到的 schedule 明显依赖 condition，不能由单一全局 time schedule 概括。

## 结果边界

- Self-consistency 比必要的 marginal consistency 更强。$\omega^\phi_{c,(s,t)}$ 不依赖 $x_t$ 或具体 $x_0$，论文明确指出它通常无法精确满足 self-consistency；这是低方差的实用 surrogate。
- 控制器是 condition- 和 step-adaptive，不是 state-adaptive，不能据此推出 event-triggered control 已解决。
- reward consistency 缓解但未消除 distribution shift；CLIP 与 FID 的实验差异即是警告。
- 当前记录对应 arXiv v1 预印本，不写成同行评审后的定论。

## 关系

- 方法页：[[wiki/methods/learn-to-guide]]；基础操作：[[wiki/concepts/classifier-free-guidance]]。
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]] 使用手工矩形窗；本文推广为 condition- 与 $(s,t)$-dependent 连续权重。
- [[wiki/methods/cfg-plus-plus|CFG++]] 改 guidance 进入一步更新的哪个分量；本文学习 scale，不做 denoise/renoise splitting。
- MS-COCO 实验直接在 Flow Matching velocity model 上训练 guidance network，是跨底座实证。

## 对我的 thesis 的启示

1. **working thesis 版本不变。** 本文强化“冻结主模型、学习低维采样控制器”，但不直接验证 editing 或 Reject-and-Skip。
2. 它只观察 $(c,s,t)$，不观察轨迹状态 $x_t$；因此是 state-aware / event-triggered 方法应在同 backbone、solver、NFE 下超过的强 baseline。
3. Flow Matching 接口已有实证，但方法不包含 trial rejection、回滚或 skip destination 搜索。

## 待调研方向

- [ ] 加入 $x_t$、conditional–unconditional disagreement、velocity curvature 或可信度统计能否改善一致性？
- [ ] learned scale 与 CFG++ operator splitting 能否组合？
- [ ] 在 text-guided editing 中能否同时改善 source fidelity 与 instruction adherence？
- [ ] 相对矩形 interval，收益有多少来自 condition adaptation，多少来自更细的 step parameterization？

## 出处

- [[raw/literature-notes/galashovLearnGuideYour2025]]
- arXiv: 2510.00815v1
