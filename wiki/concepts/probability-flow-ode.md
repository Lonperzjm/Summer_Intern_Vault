---
type: concept
title: Probability-Flow ODE
aliases: [probability flow ODE, PF-ODE, 概率流 ODE, 确定性采样 ODE]
tags: [diffusion, sde, sampling, ode, score-based]
status: active
created: 2026-05-20
updated: 2026-05-29
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/liuFlowStraightFast2022a]]"]
---

# Probability-Flow ODE

## 一句话定义

与前向扩散 SDE **共享同一族边缘分布** $\{p_t(x)\}$ 的一条**确定性**常微分方程；它把"带随机游走的扩散"替换为"无噪声的流"，因此每个初始噪声唯一决定一条样本轨迹。

$$\mathrm dx = \Big[f(x,t) - \tfrac12 g(t)^2\,\nabla_x\log p_t(x)\Big]\,\mathrm dt.$$

## 怎么来的

从 [[wiki/concepts/fokker-planck-equation|Fokker-Planck 方程]] 出发，把扩散项 $\tfrac12 g^2\Delta_x p_t$ 整理进散度算子，得到一条**连续性方程** $\partial_t p_t = -\nabla_x\!\cdot(\tilde f\,p_t)$，其中
$$\tilde f(x,t) = f(x,t) - \tfrac12 g(t)^2\nabla_x\log p_t(x).$$
连续性方程对应的确定性流就是上面的 ODE——它产生的边缘恰好是 $p_t$（与原 SDE 同），但没有扩散项。注意系数是 $\tfrac12 g^2$（反向 SDE 里是 $g^2$）。

## 三个用途

1. **确定性采样**：把训练好的 $s_\theta(x,t)\approx\nabla_x\log p_t(x)$ 代入，用黑盒 ODE 求解器（RK45、Euler 等）反向积分。步数可比 SDE 少很多。
2. **精确似然**：ODE 是连续归一化流（CNF），用 instantaneous change of variables
   $$\log p_0(x(0)) = \log p_T(x(T)) + \int_0^T \nabla\!\cdot\tilde f(x(t),t)\,\mathrm dt,$$
   可算精确 NLL——CIFAR-10 报到 **2.99 bits/dim**。
3. **可逆编码 / inversion**：ODE 双向确定，$x_0\leftrightarrow x_T$ 一一对应，给 latent 插值、语义编辑提供入口。

## 与 DDIM 的关系（重要）

[[wiki/methods/ddim|DDIM]] 的确定性采样（$\sigma=0$）正是 probability-flow ODE 的一个**离散化特例**：DDIM 论文给出的 $\mathrm d\bar x=\varepsilon_\theta(\cdot)\,\mathrm d\sigma$ 与 VP-SDE 的 PF-ODE 同源。可以说 **DDIM = diffusion 的训练 + flow 的采样**；这条 ODE 也是通往 [[wiki/concepts/flow-matching|Flow Matching]] / [[wiki/methods/rectified-flow|Rectified Flow]]（连训练也 flow 化）的桥——FM/RF 直接训练速度场，PF-ODE 则事后从 score 导出（[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]] / [[wiki/sources/liuFlowStraightFast2022a|Liu et al. 2022]]）。**进一步**：[[wiki/methods/rectified-flow|RF]] 通过 [[wiki/concepts/reflow|reflow]] 在训练阶段把 ODE 迭代拉直，相当于在 PF-ODE 这条家谱的"训练侧"加了一道直线化操作——PF-ODE 是事后导出的确定性 ODE，RF 是被主动拉直的确定性 ODE。

## 与其他概念的关系

- 与 [[wiki/concepts/predictor-corrector-sampling|PC 采样]] 并列，是 [[wiki/concepts/score-sde|Score SDE]] 的两类采样器之一（确定性 vs 随机）
- 共享边缘的依据是 [[wiki/concepts/fokker-planck-equation]]
- score 由 [[wiki/concepts/score-matching]] 训练给出
- **速度场本质**：PF-ODE 的速度场 $f-\tfrac12 g^2\nabla\log p$ 是**保守场**（VE/VP 下均可验证），而 [[wiki/concepts/flow-matching|FM]] 的速度场一般非保守——这是 PF-ODE 与 FM-ODE"同为确定性 ODE 却不等价"的根源，详见 [[wiki/comparisons/score-vs-velocity-field]]

## 在 text-guided editing 中的作用

- 确定性 + 可逆 = inversion-based 编辑的理论底座（DDIM inversion 的连续根）
- 少步采样降低反复编辑的成本；但反向积分离散误差会累积，影响 inversion 往返闭合（与 [[wiki/methods/ddim]] failure mode 对照）

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（PF-ODE 提出，精确似然）
- [[wiki/sources/songDenoisingDiffusionImplicit2022]]（DDIM 确定性采样 = 其离散化）
- Chen et al. 2018（Neural ODE / instantaneous change of variables）
