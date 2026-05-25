---
type: concept
title: Score SDE（连续时间 score-based 生成框架）
aliases: [Score SDE, "Yang Song et al. 2021", score-based SDE, 连续时间极限, VP-SDE, VE-SDE, sub-VP-SDE]
tags: [diffusion, score-based, sde]
status: active
created: 2026-05-14
updated: 2026-05-24
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]"]
---

# Score SDE（连续时间 score-based 生成框架）

> 概念主页。源：[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021, ICLR]]。
> 注：此处 "Song" 指 **[[wiki/entities/yang-song|Yang Song]]**；与 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 的 "Song"（[[wiki/entities/jiaming-song|Jiaming Song]]）不是同一人，二者均为 ICLR 2021。

## 一句话定义

把离散的加噪/去噪链取**连续时间极限**，写成一对随机微分方程：前向 SDE 把数据扩散成噪声，对应的反向 SDE（只依赖 score $\nabla_x\log p_t(x)$）把噪声还原成数据；并给出与之共享边缘分布的确定性 **probability-flow ODE**。[[wiki/methods/ncsn|SMLD]] 与 [[wiki/methods/ddpm|DDPM]] 在此框架下被统一为两类 SDE 的离散化。

## 公式链（三式串全文）

$$
\underbrace{\mathrm dx = f\,\mathrm dt + g\,\mathrm dw}_{\text{前向 SDE（加噪）}}
\;\Longrightarrow\;
\underbrace{\partial_t p_t = -\nabla\!\cdot(f p_t)+\tfrac12 g^2\Delta p_t}_{\text{Fokker-Planck（分析/训练）}}
\;\Longrightarrow\;
\underbrace{\mathrm dx=[f-g^2\nabla\log p_t]\mathrm dt+g\,\mathrm d\bar w}_{\text{反向 SDE（采样，}\mathrm dt<0)}
$$

- 第 1、3 式服务 sampling，第 2 式（[[wiki/concepts/fokker-planck-equation|Fokker-Planck]]）服务 training/分析。
- 全文灵魂：所有采样花样都是**在不改变 $\partial_t p_t$（各时刻边缘分布）的前提下换走法**。

## 三类 SDE（统一旧方法）

| SDE | $f(x,t)$ | $g(t)$ | 等价的离散方法 | 方差行为 |
|---|---|---|---|---|
| **VE-SDE** | $0$ | $\sqrt{\mathrm d[\sigma^2(t)]/\mathrm dt}$ | [[wiki/methods/ncsn\|SMLD/NCSN]] | Variance Exploding |
| **VP-SDE** | $-\tfrac12\beta(t)x$ | $\sqrt{\beta(t)}$ | [[wiki/methods/ddpm\|DDPM]] | Variance Preserving |
| **sub-VP-SDE** | $-\tfrac12\beta(t)x$ | $\sqrt{\beta(t)(1-e^{-2\int_0^t\beta})}$ | 本文新提 | 似然更优 |

## 训练：连续时间 denoising score matching

$$\theta^* = \arg\min_\theta \mathbb E_t\Big[\lambda(t)\,\mathbb E_{x(0)}\mathbb E_{x(t)\mid x(0)}\big\|s_\theta(x(t),t)-\nabla_{x(t)}\log p_{0t}(x(t)\mid x(0))\big\|_2^2\Big]$$

前向 SDE 的转移核 $p_{0t}(x(t)\mid x(0))$ 为高斯（VE/VP 闭式），条件 score 解析可写；最优网络学到的是**边缘 score**（靠 $\nabla_{\tilde x}\log p_\sigma(\tilde x)=\mathbb E_{p(x\mid\tilde x)}[\nabla_{\tilde x}\log p_\sigma(\tilde x\mid x)]$，详见 [[wiki/concepts/score-matching]]）。配方：① 选 SDE → ② 写 $p_{0t}$ → ③ 算条件 score → ④ 训 $s_\theta$ → ⑤ 构造反向 SDE 采样。

## 采样：两类求解器

- [[wiki/concepts/predictor-corrector-sampling|Predictor-Corrector]]：反向 SDE 离散步（predictor）+ 原地 Langevin（corrector）。corrector 稳态即 $p_t$，不推进时间、只提纯。论文中 PC > 单纯加倍 predictor。
- [[wiki/concepts/probability-flow-ode|Probability-flow ODE]]：$\mathrm dx=[f-\tfrac12 g^2\nabla\log p_t]\mathrm dt$，确定性、共享同一 $p_t$；解锁精确似然与可逆 inversion。

## 条件生成

$$\mathrm dx=\big\{f-g^2[\nabla\log p_t(x)+\nabla\log p_t(y\mid x)]\big\}\mathrm dt+g\,\mathrm d\bar w$$
由贝叶斯 $\nabla\log p_t(x\mid y)=\nabla\log p_t(x)+\nabla\log p_t(y\mid x)$ 而来，无需重训无条件模型——即 [[wiki/concepts/classifier-guidance|classifier guidance]] 的连续形式（其内化版为 [[wiki/concepts/classifier-free-guidance|CFG]]）。

## 与其他概念的关系

- 严格化 [[wiki/concepts/diffusion-process]] 中「small-$\beta$ ⇒ 反向高斯」的离散近似（连续极限下精确）——回应 DDPM 笔记里 🔴 的 SDE 理论疑虑
- 统一 [[wiki/concepts/score-matching]] 的离散多尺度版本与连续版本
- 与 [[wiki/concepts/langevin-dynamics]]：corrector / 反向过程是 annealed Langevin 的连续时间对应物
- [[wiki/concepts/probability-flow-ode]] 是 [[wiki/methods/ddim|DDIM]] 确定性采样的连续母体，并通往 [[wiki/concepts/flow-matching|Flow Matching]] / Rectified Flow——区别在于 FM **直接把训练目标换成回归速度场**（ODE 是训出来的本体），PF-ODE 则由训练好的 score **事后导出**（[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]）

## 在 text-guided editing 中的作用

- 提供「editing = 在反向 SDE/ODE 轨迹上注入条件」的连续视角，是理解 inversion 与确定性编辑（DDIM inversion）的理论底座
- 条件反向 SDE 把"引导项"显式加在每个 $t$——"在哪个时间步、注入多强"成为连续可调旋钮，正对 fidelity↔controllability 假设

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（Score SDE 原文，已 ingest）
- [[wiki/sources/songDenoisingDiffusionImplicit2022]] —— DDIM 的 ODE 视角是 probability-flow ODE 的离散化
- 经典：Anderson 1982（time reversal）；Vincent 2011（DSM）
