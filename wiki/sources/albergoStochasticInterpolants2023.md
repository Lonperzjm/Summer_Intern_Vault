---
type: source
title: "Stochastic Interpolants: A Unifying Framework for Flows and Diffusions"
aliases: [Stochastic Interpolants, 随机插值, "Albergo, Boffi & Vanden-Eijnden 2023", "Albergo & Vanden-Eijnden 2023"]
tags: [stochastic-interpolants, flow-matching, diffusion-bridge, sde, ode, generative-model, foundational]
status: stable
created: 2026-06-01
updated: 2026-06-01
raw: "[[raw/articles/2303.08797v4.pdf]]"
authors: [Michael S. Albergo, Nicholas M. Boffi, Eric Vanden-Eijnden]
venue: "JMLR 26 (2025) / arXiv 2303.08797"
year: 2023
arxiv: "2303.08797"
---

# Stochastic Interpolants: A Unifying Framework for Flows and Diffusions

> 原文 PDF：[[raw/articles/2303.08797v4.pdf]] · arXiv [2303.08797](https://arxiv.org/abs/2303.08797) · Albergo, Boffi & Vanden-Eijnden（NYU Courant，JMLR 2025）
> 概念主页：[[wiki/concepts/stochastic-interpolants]]。本页是该框架在 vault 的源摘要。

## 一句话

把"**设计连接两分布的桥 $\rho(t)$**"与"**怎么采样它**"彻底解耦：自由选一条 **stochastic interpolant** $x_t=\alpha_t x_0+\beta_t x_1+\gamma_t z$ 直接连接**任意两个分布** $\rho_0,\rho_1$（有限时间精确到达两端），其密度同时满足一阶 transport equation（→ ODE）与**一族可调噪声系数的 Fokker-Planck**（→ SDE）；drift（velocity）与 score 都是**简单二次目标的唯一最小值**，可从数据回归。[[wiki/concepts/score-sde|score diffusion]]、[[wiki/methods/rectified-flow|rectified flow]] 是其特例，显式优化插值时**还原 [[wiki/synthesis/bridge-sde-editing-landscape|Schrödinger Bridge]]**。

## Motivation

既有生成模型把"加噪过程/路径"与"采样动力学"耦在一起（diffusion 先定一条 noise→data SDE 再反解）。作者要一个更一般、更模块化的框架：**先自由设计 probability path，再独立选 ODE 还是 SDE（噪声量可调）去采样**，并用最简单的二次回归学 drift——把 flow 与 diffusion 收进同一套语言。

## Method

### 1. Stochastic interpolant（核心定义）

$$
x_t=\alpha(t)x_0+\beta(t)x_1+\gamma(t)z,\quad t\in[0,1],\quad x_0\sim\rho_0,\ x_1\sim\rho_1,\ z\sim\mathcal N(0,I)\ \text{独立}
$$
边界 $\alpha_0{=}\beta_1{=}1,\ \alpha_1{=}\beta_0{=}\gamma_0{=}\gamma_1{=}0$ ⇒ $x_0\sim\rho_0,\ x_1\sim\rho_1$ **精确到达两端**。例（1.1）：$x_t=(1-t)x_0+tx_1+\sqrt{2t(1-t)}\,z$。**$\rho_0,\rho_1$ 任意**（data↔data，不限 noise↔data），latent $z$ 给桥"塑形"。

### 2. 一个密度，多种动力学（关键解耦）

> 同一条 $x_t$ 的边缘密度 $\rho(t)$ 可由**很多过程**实现——一条 ODE、一族 forward/backward SDE，其**扩散系数 $\epsilon(t)$ 可任意调**。

$\rho(t)$ 满足一阶 transport equation（确定流）+ 一族 Fokker-Planck（随机流）。于是：
- **ODE（probability flow）**：$\dot x=b(t,x)$，velocity $b$；
- **SDE**：$\mathrm dx=b\,\mathrm dt+\text{（score 项，系数 }\epsilon(t)\text{）}+\sqrt{2\epsilon(t)}\,\mathrm dW$，**$\epsilon$ 是噪声旋钮**。

两者**共享同一 $\rho(t)$、同一 velocity + score 估计**，只是路径不同。

### 3. 二次目标学 drift / score

velocity $b$、score $\nabla\log\rho(t)$（及 denoiser $\eta_z$）都是**简单二次（最小二乘）目标的唯一最小值**，用 $\rho_0,\rho_1,\mathcal N(0,I)$ 的样本即可估。提出了 interpolant density 的 **score 的新目标**。

### 4. Likelihood control（ODE vs SDE 的关键差异）

- **SDE-based**：回归 drift 即可**控制 likelihood**。
- **ODE-based**：回归 drift **不够**，还须额外最小化一个 **Fisher divergence**——故确定性模型的似然控制更严苛。可**最优调** $\epsilon(t)$。

## Results

理论为主，数值佐证：
- **2D / 128D 高斯混合**：系统比较确定性（ODE）vs 随机（SDE）生成——存在"随机动力学在样本质量上优于确定动力学"的 regime（呼应 diffusion 经验）。
- **图像（Oxford flowers 128²，mirror interpolant，Fig 15）**：用 ODE 时输出 = 输入；**用 SDE（$\epsilon$ 控制）时能从同一输入生成新样本**——"original image is resampled to a proximal flower not seen in the dataset"，$\epsilon$ 越大偏离越多。🟣 这就是"**同一输入 + 随机性 → 可控多样性变体**"的机制，直接对应你 thesis 的 diversity 旋钮（见下）。

## 关系（与已有 wiki 的关联）

- **本页是 [[wiki/concepts/stochastic-interpolants]] 的源**（该页由 draft 升为 active）。
- **统一/收编**：[[wiki/concepts/score-sde|score-based diffusion]]、stochastic localization、denoising、[[wiki/methods/rectified-flow|rectified flow]] 均为特例（§5）；显式优化插值 → [[wiki/synthesis/bridge-sde-editing-landscape|Schrödinger Bridge]]（§3.4）。
- **与 [[wiki/concepts/flow-matching|Flow Matching]]**：同期并行、互补——FM 给 conditional velocity 的 simulation-free 训练，SI 把"自由选 path + 选 ODE/SDE + 可调噪声"系统化并加 score/likelihood 理论。
- **与 [[wiki/methods/ddbm|DDBM]] / [[wiki/concepts/diffusion-bridge|扩散桥]]**：SI 是"**bridge 不依赖参考扩散**"的一般框架；DDBM 反而把桥又绑回 VP/VE 扩散以复用其工程。SI 用的是 interpolant + 二次目标，DDBM 用 Doob h + denoising bridge score matching（不同 loss）。
- **数学底座**：[[wiki/concepts/fokker-planck-equation|Fokker-Planck]]（一族可调系数版）、[[wiki/concepts/probability-flow-ode|probability-flow ODE]]、[[wiki/concepts/score-matching]]。
- **人物 / 机构**：[[wiki/entities/michael-albergo]]、Nicholas Boffi、[[wiki/entities/eric-vanden-eijnden]]；[[wiki/entities/nyu-courant]]。

## 对我的 thesis 的启示

> ⚠️ **直接结论（对你选题的判决，已与你确认）**：你"让 SDE bridge 脱离 diffusion 框架"的想法 = **本框架本身**。SI 已经做到：自由选插值、不从扩散推、data↔data、ODE/SDE 双实现、可调噪声、还原 SB。所以**作为理论方向它被占满，应放弃**（[[research/ideas]] 已 park）。

但本文 **Conclusion 自己指的 future application 给了你 editing 的合法入口**——原文点名："inverse problems such as image **inpainting** and **super-resolution**"、forecasting、scientific sampling。即：
- 框架是 frontier、但**应用到 text-guided editing 仍是开放的**（SI 只演示了无条件生成 + 图像 resample 变体，没做 text-driven、target-aware 编辑）。
- Fig 15 的"SDE + $\epsilon$ → 同输入多变体"正是你可借的**可控多样性**机制。于是 thesis 候选回到：**用 SI 的自由插值 + 可调 $\epsilon$，给 text editing 一个 fidelity↔diversity 旋钮**——这是**应用贡献**（开放）而非理论贡献（被占）。⚠️ 仍须 sweep 确认"stochastic / diverse text editing"没被占。

（按惯例未改 [[wiki/overview]] / [[research/thesis]]，候选留你定。）

## Open questions / 待追

- [ ] 🟢 **interpolant 设计空间**：原文只详述 mirror / one-sided 两类，明言"much broader space of possible designs"——新设计是公开的缝（但偏理论）。
- [ ] $\epsilon(t)$ 最优调度在**编辑/条件**场景下的形式（原文是无条件）。
- [ ] SI 上做**条件 / text-guided** 生成的接口（原文未涉及条件注入）。
- [ ] one-sided interpolant（$\rho_0$ 为高斯）退回 diffusion 的精确对应（§3.2）。
