---
type: source
title: "Score-Based Generative Modeling through Stochastic Differential Equations"
aliases: [Score SDE 原文, "Yang Song et al. 2021", SDE 统一框架]
tags: [diffusion, score-based, sde, foundational]
status: stable
created: 2026-05-20
updated: 2026-05-20
raw: "[[raw/literature-notes/songScoreBasedGenerativeModeling2021]]"
authors: [Yang Song, Jascha Sohl-Dickstein, Diederik P. Kingma, Abhishek Kumar, Stefano Ermon, Ben Poole]
venue: ICLR 2021 (Oral / Outstanding Paper)
year: 2021
arxiv: "2011.13456"
---

# Score-Based Generative Modeling through Stochastic Differential Equations

> 文献笔记：[[raw/literature-notes/songScoreBasedGenerativeModeling2021]] · arXiv [2011.13456](http://arxiv.org/abs/2011.13456) · Song et al. 2021
> 注：此 "Song" 指 **[[wiki/entities/yang-song|Yang Song]]**；与 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 的 "Song"（[[wiki/entities/jiaming-song|Jiaming Song]]）不是同一人，二者均为 ICLR 2021。

## 一句话

把离散的加噪/去噪链取**连续时间极限**，统一写成一对随机微分方程：前向 SDE 把数据扩散成噪声，反向 SDE（**只依赖 score** $\nabla_x\log p_t(x)$）把噪声还原成数据；并据此导出 predictor-corrector 采样、确定性 probability-flow ODE（带精确似然）与基于贝叶斯的条件生成。[[wiki/methods/ncsn|SMLD/NCSN]] 与 [[wiki/methods/ddpm|DDPM]] 被证明是同一框架下两类 SDE 的离散化。

## Motivation

作者要解决的是**碎片化**：当时 score-based 生成有两条看似独立的主线——
- [[wiki/methods/ncsn|SMLD（NCSN, Song & Ermon 2019）]]：多噪声尺度 [[wiki/concepts/score-matching|denoising score matching]] + [[wiki/concepts/langevin-dynamics|annealed Langevin]] 采样；
- [[wiki/methods/ddpm|DDPM（Ho et al. 2020）]]：变分去噪链 + [[wiki/concepts/epsilon-parameterization|ε-prediction]]。

二者经验上都很强但理论上各说各话，且都受限于**有限个离散噪声尺度**。本文主张：把噪声尺度数推到连续极限（$N\to\infty$），整个过程就是一条 SDE，于是 (i) 两条主线统一为同一框架的两种离散化；(ii) 可以借用成熟的数值 SDE/ODE 求解器换出新采样器；(iii) 解锁精确似然与可控生成。

## Method

### 1. 前向 SDE 与 Fokker-Planck（核心公式链）

前向加噪 = 一条 Itô SDE：
$$\mathrm dx = f(x,t)\,\mathrm dt + g(t)\,\mathrm dw.$$
其边缘密度 $p_t(x)$ 的演化由 **[[wiki/concepts/fokker-planck-equation|Fokker-Planck 方程]]** 给出：
$$\frac{\partial p_t(x)}{\partial t} = -\nabla_x\!\cdot\!\big(f(x,t)p_t(x)\big) + \tfrac12 g(t)^2\,\Delta_x p_t(x).$$
反向（[Anderson 1982] 的 time-reversal）仍是一条 SDE，**只依赖 score**：
$$\mathrm dx = \big[f(x,t) - g(t)^2\,\nabla_x\log p_t(x)\big]\,\mathrm dt + g(t)\,\mathrm d\bar w,\qquad \mathrm dt<0.$$

> **本页贯穿全文的"灵魂"（采纳用户 takeaway #1 的 framing）**：第 1、3 式服务 sampling，第 2 式服务 training；后面所有花样（PC corrector、probability-flow ODE）都是**在不改变 Fokker-Planck（即不改变各时刻边缘 $p_t$）的前提下换一种走法**。

### 2. 两类 SDE 统一（采纳用户 takeaway #2 的概率图像）

- **VE-SDE**（Variance Exploding）= [[wiki/methods/ncsn|SMLD/NCSN]] 的连续极限：$f=0,\ g(t)=\sqrt{\tfrac{\mathrm d[\sigma^2(t)]}{\mathrm dt}}$，方差随 $t$ 爆炸增大。
- **VP-SDE**（Variance Preserving）= [[wiki/methods/ddpm|DDPM]] 的连续极限：$f=-\tfrac12\beta(t)x,\ g(t)=\sqrt{\beta(t)}$，方差保持有界。每个 $x$ 朝 0 收缩 + 随机游走（见用户的概率演化图）。
- **sub-VP-SDE**：本文新提，似然更优。

![[songScoreBasedGenerativeModeling2021-1779104321502.webp]]

### 3. 训练：连续时间 denoising score matching（采纳用户 takeaway #3）

目标是让 $s_\theta(x,t)$ 匹配 $\nabla_x\log p_t(x)$。关键恒等式（[[wiki/concepts/score-matching|DSM]]）使**条件 score 作监督、最优解却是边缘 score**：
$$\nabla_{\tilde x}\log p_\sigma(\tilde x) = \mathbb E_{p(x\mid\tilde x)}\big[\nabla_{\tilde x}\log p_\sigma(\tilde x\mid x)\big]\quad(\text{贝叶斯}).$$
统一训练目标：
$$\theta^* = \arg\min_\theta \mathbb E_t\Big[\lambda(t)\,\mathbb E_{x(0)}\,\mathbb E_{x(t)\mid x(0)}\big\|s_\theta(x(t),t)-\nabla_{x(t)}\log p_{0t}(x(t)\mid x(0))\big\|_2^2\Big].$$
因为前向 SDE 的转移核 $p_{0t}(x(t)\mid x(0))$ 是高斯（VE/VP 都有闭式），条件 score 解析可写，训练即对噪声的加权最小二乘——与 $\varepsilon$-prediction 在尺度因子下等价。

**通用配方**（采纳用户 takeaway #4）：① 选前向 SDE → ② 写转移核 $p_{0t}$ → ③ 算条件 score 作目标 → ④ 训 $s_\theta$ → ⑤ 采样时构造反向 SDE。

### 4. 采样：三种走法（采纳用户 takeaway #5）

- **Predictor-Corrector（PC）**：predictor = 反向 SDE 的离散步；corrector = 在**同一时刻**跑 $m$ 步 [[wiki/concepts/langevin-dynamics|Langevin MCMC]]。corrector 的稳态是 $p_t$ 本身，$\partial p_t/\partial t=0$，故**不推进时间、只把样本拉回该时刻的高概率流形**，修正离散化/随机扰动带来的偏差。论文报告 PC 优于"把 predictor 步数翻倍但不加 corrector"（如 P2000 基线）。
- **[[wiki/concepts/probability-flow-ode|Probability-flow ODE]]**：与前向 SDE 共享同一族边缘 $p_t$ 的**确定性** ODE
  $$\mathrm dx = \big[f(x,t) - \tfrac12 g(t)^2\,\nabla_x\log p_t(x)\big]\,\mathrm dt.$$
  → 确定性采样（DDIM 的连续母体）、latent 编码/插值、用 instantaneous change of variables 做**精确似然**计算。

### 5. 条件生成（采纳用户 takeaway #6）

条件反向 SDE 由贝叶斯拆分得到，**无需重训无条件模型**：
$$\mathrm dx = \Big\{f(x,t) - g(t)^2\big[\underbrace{\nabla_x\log p_t(x)}_{\text{score network}} + \underbrace{\nabla_x\log p_t(y\mid x)}_{\text{引导项}}\big]\Big\}\mathrm dt + g(t)\,\mathrm d\bar w.$$
引导项可由分类器、观测前向模型或启发式给出——这正是 **classifier guidance** 的形式（[[wiki/concepts/classifier-free-guidance]] 的前身），并被用于 class-conditional 生成、inpainting、colorization 等逆问题。

## Results

- **[[wiki/benchmarks/cifar10|CIFAR-10]] 无条件**：Inception Score **9.89**、FID **2.20**（当时 SOTA）；likelihood **2.99 bits/dim**（probability-flow ODE 算的，competitive）。
- **首次**用 score-based 模型做到 **1024×1024** 高保真生成。
- **PC > 单纯加倍 predictor**：相同算力下 corrector 带来一致提升。
- **逆问题**：class-conditional 生成、inpainting、colorization 均用同一条件反向 SDE 求解，无需为每个任务重训。

## 关系（与已有 wiki 的关联）

- **本页是 [[wiki/concepts/score-sde]] 概念条目的源**——该 stub 由本次 ingest 升级为完整页。
- 统一了 [[wiki/methods/ncsn|NCSN（VE-SDE）]] 与 [[wiki/methods/ddpm|DDPM（VP-SDE）]]，严格化 [[wiki/concepts/diffusion-process]] 中「small-$\beta$ ⇒ 反向高斯」的离散近似（连续极限下精确）。
- 训练侧把 [[wiki/concepts/score-matching|denoising score matching]] 推广到连续时间；采样侧 corrector 用 [[wiki/concepts/langevin-dynamics|Langevin]]。
- [[wiki/concepts/probability-flow-ode|Probability-flow ODE]] 是 [[wiki/methods/ddim|DDIM]] 确定性采样（$\sigma=0$）的连续母体——DDIM 的 ODE $\mathrm d\bar x=\varepsilon_\theta\,\mathrm d\sigma$ 与之同源。
- 条件反向 SDE 是 [[wiki/concepts/classifier-free-guidance|CFG]]/classifier guidance 的连续理论底座。
- 人物：[[wiki/entities/yang-song]]（一作）、[[wiki/entities/stefano-ermon]]（资深作者）；机构 [[wiki/entities/stanford]] + Google Brain。

## 对我的 thesis 的启示

- **强化 overview 推论 1（可变性光谱）**：本文把"训练目标可演化、范式不变"讲到极致——同一个 score 网络可被三种不同采样器（SDE / PC / ODE）复用，采样器属于光谱中"研究杠杆/介入方式"一档，代价低、不动训练。
- **强化 overview 推论 3（采样加速是开放赛道）**：probability-flow ODE 把"确定性快采样 + 精确似然"统一进同一框架，是 DDIM 之上的连续化总纲；编辑场景反复采样，受益直接。
- **对推论 2（介入时间步 trade-off）有新弹药**：条件反向 SDE 显式把"引导项"$\nabla\log p_t(y\mid x)$ 加在每个 $t$ 上——"在哪个时间步注入条件、注入多强"成为连续可调的旋钮，正对 fidelity↔controllability 假设。
- **inversion 视角**：probability-flow ODE 的可逆性是 DDIM inversion 的连续根，是 inversion-based 编辑的理论底座。

> 拟据此微调 [[wiki/overview]] working thesis（推论 1/3 加"采样器可换"的干净样本，推论 2 加"连续引导旋钮"）——已单独做成 diff 待用户确认，未直接改动。

## Open questions / 待追

- VE vs VP vs sub-VP 在**编辑任务**上的优劣差异（本文只比生成质量/似然）。
- PC 的 corrector 步数 $m$、Langevin 步长 $\varepsilon_i$ 的调参与编辑保真度的关系。
- probability-flow ODE 反向积分的离散误差如何影响 inversion 往返闭合——与 [[wiki/methods/ddim]] failure mode 对照。
- 与 Flow Matching / Rectified Flow 的精确关系（ODE 训练 vs 事后导出）——待 ingest FM 原文。
