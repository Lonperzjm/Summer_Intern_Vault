---
type: concept
title: Flow Matching（FM）
aliases: [Flow Matching, FM, 流匹配]
tags: [flow-matching, cnf, generative-model, ode]
status: active
created: 2026-05-24
updated: 2026-06-01
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]", "[[wiki/sources/liuFlowStraightFast2022a]]", "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# Flow Matching（FM）

> 概念主页。源：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023, ICLR]]。
> ⚠️ 时间约定：FM 中 $t=0$ 噪声、$t=1$ 数据，生成沿 $t:0\to1$ 正向；与本 vault 的 [[wiki/methods/ddpm|DDPM]]/[[wiki/concepts/score-sde|Score SDE]]（$t:0\to T$ 数据→噪声）**相反**。

## 一句话定义

不构造加噪 SDE，而是**直接选一条 probability path $p_t$（噪声↔数据）、用网络回归它的生成向量场 $v_t$**，再沿 ODE $\dot x=v_t(x)$ 生成。这给出 simulation-free（训练时不解 ODE）地训练 [[wiki/concepts/continuous-normalizing-flow|CNF]] 的办法。

## 与 score-based 的根本分工

| | score-based（[[wiki/concepts/score-sde]]） | Flow Matching |
|---|---|---|
| 先验对象 | 一条加噪 **SDE**（$v=f$ 解析已知） | 一条 **probability path** $p_t$ |
| 网络学什么 | score $\nabla_x\log p_t$ | 速度场 $v_t(x)$ |
| 采样 | 反向 SDE / [[wiki/concepts/probability-flow-ode\|PF-ODE]] | ODE $\dot x=v_t$ |
| 关于 $p(x)$ 的知识 | 在 score 里 | **全在 $v(x,t)$ 里** |

> 一句话延续 vault 既有提法：**DDIM/PF-ODE = diffusion 的训练 + flow 的采样；FM = 连训练也 flow 化**。

> 📐 **更深一层（对象本质）**：score $\nabla\log p_t$ 必是**保守场**（梯度），$v_t$ 一般是**非保守场**（写不成 $\nabla(\cdot)$）——两者只在固定高斯路径下可线性互转。这是 FM 更一般、OT 路径能"走直线"的数学根源，详见 [[wiki/comparisons/score-vs-velocity-field]]。

## 朴素目标与它的麻烦

$$\mathcal L_{\mathrm{FM}}(\theta)=\mathbb E_{t,\,p_t(x)}\big\|v_t(x;\theta)-u_t(x)\big\|^2.$$
但边缘 $p_t,u_t$ 无闭式、不可采样 → intractable。解法是 [[wiki/concepts/conditional-flow-matching|Conditional Flow Matching]]：用 per-example 条件场作回归目标，**梯度与上式相同**（同 [[wiki/concepts/score-matching|DSM]] 套路）。

## 自洽性判据：连续性方程

$(p_t,v_t)$ 相容 ⟺ 满足
$$\frac{\partial p_t}{\partial t}+\nabla\!\cdot(p_t v_t)=0,$$
即 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]] 在无扩散项（$g=0$）时的退化——FM 是确定性流，没有 $\tfrac12 g^2\Delta p$。

## 为什么重要

1. **路径成了可设计对象**：diffusion 路径只是 Gaussian 路径族的特例；可换上 [[wiki/concepts/optimal-transport-path|OT 直线路径]]，训练/采样更快。
2. **真正换掉了训练目标**却不破坏"迭代生成 + 预测速度场 + 沿链注入条件"的范式——是 [[wiki/overview]] 可变性光谱里"训练目标可演化、范式不变"的关键样本。
3. **采样省**：用现成自适应 ODE solver，NFE 低、训练期采样成本恒定。

## 与其他概念的关系

- 训练技巧：[[wiki/concepts/conditional-flow-matching]]（核心）
- 载体：[[wiki/concepts/continuous-normalizing-flow]]
- 路径实例：[[wiki/concepts/optimal-transport-path]]、diffusion 路径（[[wiki/methods/ddpm]]/[[wiki/methods/ncsn]]）
- 采样近亲：[[wiki/concepts/probability-flow-ode]]（同为确定性 ODE，但 score 事后导出 vs FM 直接训练）
- 自洽判据：[[wiki/concepts/fokker-planck-equation]]（无扩散退化）

## 在 text-guided editing 中的作用

- FM 是 SD3 / FLUX 等 [[wiki/methods/rectified-flow|rectified-flow]] 一线的训练底座；这些模型上的编辑方法（如 RF-Inversion）继承"在 ODE 链上注入条件"的范式。
- ⚠️ FM 的 $t$ 与 diffusion 的 $t$ 方向相反，做"介入时间步"分析时须先统一坐标。

## 出处与引用

- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（FM 原文）
- 并行工作：[[wiki/sources/liuFlowStraightFast2022a|Liu et al. 2022 (Rectified Flow)]]（已 ingest 2026-05-26；公式上 RF = FM-OT 路径取 $\sigma_{\min}=0$ + 任意 coupling 接口 + [[wiki/concepts/reflow|reflow]]）、[[wiki/concepts/stochastic-interpolants|Stochastic Interpolants]]（Albergo & Vanden-Eijnden，原文待 ingest——是"固定 interpolant + 选 diffusion 系数"统一 flow 与 diffusion 的框架）
- **bridge ODE vs bridge SDE**：FM/RF 是**确定 ODE**地连接两端分布（学速度场）；[[wiki/methods/ddbm|DDBM]] 是**随机 SDE** 版的 [[wiki/concepts/diffusion-bridge|扩散桥]]（学 score、[[wiki/concepts/doob-h-transform|Doob h]] 钉端点）。DDBM 论文称"unifies OT-FM"，但其正文表明这只是 noiseless 极限 $c\to0$ + 特定 VE schedule 的**有条件约化**（非严格特例）；两者真正的统一归宿是 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]，且 DDBM 用的是不同的 denoising bridge score-matching loss
