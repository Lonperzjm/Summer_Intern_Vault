---
type: concept
title: Fokker-Planck 方程（前向 Kolmogorov）
aliases: [Fokker-Planck, FPE, forward Kolmogorov equation, 边缘分布演化]
tags: [diffusion, sde, score-based]
status: active
created: 2026-05-20
updated: 2026-05-20
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
---

# Fokker-Planck 方程

## 一句话定义

给定一条 Itô SDE $\mathrm dx=f(x,t)\,\mathrm dt+g(t)\,\mathrm dw$，其样本的**边缘密度** $p_t(x)$ 随时间演化所满足的偏微分方程：

$$\frac{\partial p_t(x)}{\partial t} = -\nabla_x\!\cdot\!\big(f(x,t)\,p_t(x)\big) + \tfrac12 g(t)^2\,\Delta_x p_t(x).$$

它把"单条随机轨迹的微观规则（SDE）"翻译成"整体概率云的宏观流动"。

## 为什么它是 Score SDE 的"灵魂"

在 [[wiki/concepts/score-sde|Score SDE]] 框架里，FPE 是判断"两种采样走法是否等价"的**不变量**：只要一个过程产生的 $p_t(x)$ 满足同一条 FPE（即各时刻边缘分布相同），它就和原前向 SDE"在分布意义上"一致，可互换使用。

由此推出全文三件法宝：

1. **反向 SDE**：要让时间倒流后边缘仍是 $p_t$，反向漂移必须含 score 项
   $$\mathrm dx=\big[f(x,t)-g(t)^2\nabla_x\log p_t(x)\big]\mathrm dt+g(t)\,\mathrm d\bar w.$$
2. **[[wiki/concepts/probability-flow-ode|Probability-flow ODE]]**：把 FPE 改写成连续性方程 $\partial_t p_t=-\nabla\!\cdot(\,\tilde f p_t)$ 的形式，可得一条**无扩散项**的确定性 ODE，共享同一族 $p_t$
   $$\mathrm dx=\big[f(x,t)-\tfrac12 g(t)^2\nabla_x\log p_t(x)\big]\mathrm dt.$$
3. **[[wiki/concepts/predictor-corrector-sampling|PC corrector]]**：在固定 $t$ 跑 [[wiki/concepts/langevin-dynamics|Langevin]]，其稳态就是 $p_t$，对应 $\partial_t p_t=0$——**不推进时间、只把样本拉回该时刻的高概率区**，因此不破坏 predictor 已建立的时间分布。

> 记忆锚点（采纳用户 takeaway #1）：第 1、3 式（反向 SDE / ODE）服务 sampling，FPE 服务 training/分析；所有采样花样都是"**在不改变 $\partial p_t/\partial t$ 的前提下换走法**"。

## 与其他概念的关系

- 描述 [[wiki/concepts/diffusion-process]] 前向链的连续时间极限
- 是 [[wiki/concepts/probability-flow-ode]] 与 [[wiki/concepts/predictor-corrector-sampling]] 成立的共同依据
- [[wiki/concepts/langevin-dynamics]] 的稳态分布即 FPE 的 $\partial_t p=0$ 解

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（用 FPE 串起反向 SDE / ODE / PC）
- 经典 SDE 理论：Anderson 1982（time reversal）、Øksendal《Stochastic Differential Equations》
