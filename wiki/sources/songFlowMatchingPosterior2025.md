---
type: source
title: "FMPS: Flow Matching Posterior Sampling (Training-free Conditional Generation for Flow Matching)"
aliases: [FMPS, "Song et al. 2025", Flow Matching Posterior Sampling]
tags: [flow-matching, guidance, training-free, clean-estimate, energy-guidance, posterior-sampling]
status: stable
created: 2026-06-29
updated: 2026-06-29
raw: "[[raw/literature-notes/songFlowMatchingPosterior2025]]"
authors: [Kaiyu Song, Hanjiang Lai, Yan Pan, Kun Yue, Jian Yin]
venue: preprint (arXiv 2024-11)
year: 2025
arxiv: "2411.07625"
---

# FMPS: Flow Matching Posterior Sampling

> 文献笔记：[[raw/literature-notes/songFlowMatchingPosterior2025]] · arXiv [2411.07625](http://arxiv.org/abs/2411.07625) · Song et al. · SYSU
> **这篇直接占了 [[research/ideas]] energy-guidance 候选的①轴**：把 FreeDoM 式 clean-estimate 引导**搬到 flow matching**，并把"一步 $\hat x_0=x_t-tv$ 贵但准 / 反解 $\hat x_0$ 便宜但糙"这个①的核心权衡**做成了两个实现**。

## 一句话

Flow matching 的网络出**速度场**、没有显式 score，所以 DPS/FreeDoM 那套靠 score 的 posterior-sampling 引导**用不了**。FMPS 用一个命题把**速度场改写成 surrogate score**，从而把 training-free 条件引导接到 FM 上；再用 FreeDoM 式 $\hat x_0$ 距离能量，给出 quality 版和 efficiency 版两种实现。

## Motivation

- training-free 条件生成（[[wiki/methods/freedom|FreeDoM]]/DPS）靠 **posterior sampling**，需要无条件模型的**显式 score** $\nabla_{x_t}\log p_t$。
- **flow matching 只有速度场 $v_\theta$、没有 score** → 直接搬不过来。此前 FM 上的近似 posterior sampling **只能做线性逆问题**。
- FMPS 把适用范围**扩到一般条件**（任意可微距离能量）。

## Method

### 1. 速度↔score 桥（Proposition 1，🟡 p.5）
前向路径 $x_t=a_t x_0+b_t\epsilon$，命题把速度场写成 score 形式：
$$v_\theta(x_t,t)=\frac{\dot b_t}{b_t}x_t+\beta_t\,\nabla_{x_t}\log p_t(x_t),\qquad \beta_t=a_t\big(\dot a_t-\tfrac{\dot b_t}{b_t}a_t\big).$$
**这是全篇关键**——把"FM 无 score"补上一个 surrogate score，DPS/FreeDoM 那套就接得上了。
> 🔴 **用户批注（p.5）："a,b 反了"** ——怀疑命题里 $a_t,b_t$ 系数写反/约定不一致，待对 PDF 原式核（affine-path 的 $\dot a/a$ vs $\dot b/b$ 系数各家约定不同，可能是 typo 也可能是约定差异）。

### 2. 引导后的速度场
贝叶斯 $\nabla\log p_t(x_t\mid c)=\nabla\log p_t(x_t)+\nabla\log p(c\mid x_t)$，配 FreeDoM 式距离近似 $\nabla\log p(c\mid x_t)\approx-\nabla_{x_t}D(\hat x_{0\mid t},c)$，得
$$\boxed{\,v_{\text{guided}}=v_\theta-r\beta_t\,\nabla_{x_t}D(\hat x_{0\mid t},c)\,}$$

### 3. 两种 $\hat x_{0\mid t}$（= ①轴的两端，🟡 p.7）
| | **FMPS-gradient**（quality） | **FMPS-free**（efficiency） |
|---|---|---|
| $\hat x_{0\mid t}$ | 一步 Euler：$x_t-t\,v_\theta$ | 前向反解：$(x_t-b_t x_1)/a_t$ |
| 梯度 | $\big(I-t\tfrac{\partial v_\theta}{\partial x_t}\big)^\top\nabla_{\hat x_0}D$ | $\tfrac{1}{a_t}\nabla_{\hat x_0}D$ |
| 取舍 | **准，但要反传过 $v_\theta$（贵）** | **便宜，但估计糙；$t=1$ 时 $a_t=0$ 首步不能用** |

### 4. 归一化（③ energy→guidance 的 trick，🟡 p.7）
把引导项缩到与速度同量级：
$$g^1=\frac{\lVert v_\theta\rVert_2}{\lVert\nabla_{x_t}D\rVert_2}\nabla_{x_t}D,\qquad v_{\text{guided}}=v_\theta-r\beta_t g^1.$$
一句话流程：$x_t\to\hat x_{0\mid t}\to D(\hat x_{0\mid t},c)\to\nabla_{x_t}D\to v_\theta-r\beta_t g^1$。

## Results

在多种条件生成任务上质量优于 SOTA training-free 方法（abstract）；验证了把 posterior sampling 接到 FM 的通用性。具体 benchmark/数字待对 PDF 补。

## 关系

- **= FreeDoM 的 flow 版**：[[wiki/methods/freedom|FreeDoM]] 在 diffusion（有 score）上做；FMPS 补上"FM 无 score"的桥（Prop 1）把同一套搬到 flow。同属 [[wiki/concepts/training-free-guidance|training-free guidance]]，框架见 [[wiki/concepts/conditional-diffusion]]。
- **$\hat x_0=x_t-tv$**：FM 版 [[wiki/concepts/tweedie-formula|Tweedie/后验均值]]（= $\mathbb E[x_0\mid x_t]$）。
- **③ 对照**：FMPS-free 走"便宜雅可比"、$g^1$ 归一化走"原理化步长"、引用 MPGD（🟡 p.3）走"流形避梯度"——③ 的三种省法这篇都沾。
- 方法页：[[wiki/methods/fmps]]。
- 作者/机构：Kaiyu Song、Hanjiang Lai、Yan Pan、Kun Yue、Jian Yin（中山大学 SYSU）。

## 对我的 thesis 的启示

> 🟣 这篇对 [[research/ideas]] energy-guidance 候选是**减分项**——它把①轴坐实占了。

- **①「RF/FM 的 $\hat x_0=x_t-tv$ + clean-estimate 引导」= FMPS 已做**，而且**两种实现正好覆盖你想探的"贵准 vs 便宜糙"权衡**（FMPS-gradient / FMPS-free）。你那条①不仅"被占"，连子问题（要不要反传雅可比）都被人列出来比过了。
- **③ 的归一化步长 $g^1$ 也被这篇做了**——③ 在 flow 上的空位又小一块。
- 净效果：候选三段里 ① 彻底埋（FMPS）、③ 再缩；**只剩 ②结构化 E**（且 ② 另有 TtfDiffusion/DICE 占）。整条候选公开文献无 carve 的判断进一步坐实，存活仅靠导师在研线。
- 不动 [[wiki/overview]] working thesis 版本号。

## 我的 takeaways

1. FM 无显式 score → Prop 1 把速度场写成 surrogate score，桥到 posterior sampling。
2. 引导速度 $v_{\text{guided}}=v_\theta-r\beta_t\nabla_{x_t}D(\hat x_{0\mid t},c)$。
3. 两种 $\hat x_0$：一步 Euler（准/贵，反传 $v_\theta$）vs 前向反解（便宜/糙，$t=1$ 不可用）。
4. $g^1$ 归一化把引导缩到速度量级（③ 的步长 trick）。
5. 对我：①被它占死，连"反传雅可比 vs 不反传"的取舍都已公开比过。

## Open questions / 待追（🔵）

- [ ] 核对 Prop 1 的 $a_t,b_t$ 系数（用户 🔴 疑写反）
- [ ] FMPS-gradient vs FMPS-free 的实测质量-成本曲线（对②/③有没有可借的诊断协议）
- [ ] 它和 [[wiki/methods/freedom|FreeDoM]]、TFG-Flow 在 benchmark 上的相对位置
