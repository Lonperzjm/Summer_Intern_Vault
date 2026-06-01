---
type: concept
title: Doob's h-transform
aliases: [Doob's h-transform, Doob h-transform, h-transform, h 变换, 条件化漂移]
tags: [sde, stochastic-process, diffusion-bridge, conditioning]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# Doob's h-transform

> 概念页。源：[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]]。严格推导（经生成元）见 [[wiki/concepts/infinitesimal-generator]] / [[raw/notes/生成元方法对于SDE]]。

## 一句话定义

把一个**普通扩散过程**改造成"**条件于固定终点** $X_T=y$"的过程的标准工具：只需在原漂移上加一个"终点吸引项"
$$
\mathrm dX_t=\big[f(X_t,t)+g(t)^2\,\nabla_x\log h(X_t,t)\big]\mathrm dt+g(t)\,\mathrm dW_t,\qquad
h(x,t)=p(X_T=y\mid X_t=x).
$$
$\nabla\log h$ 指向"当前位置微动后、未来到达 $y$ 的概率密度增加最快"的方向。

## 直觉

- $h(x,t)$ 是"从 $(x,t)$ 出发能落到 $y$ 的似然"。乘上 $h$ 再归一，等于把所有**最终到 $y$** 的样本路径挑出来重新加权。
- 加权后的过程仍是一个扩散过程，扩散系数不变（仍是 $g$），只有漂移多了 $g^2\nabla\log h$——这是生成元 $\mathcal L^h=\tfrac1h\mathcal L(h\cdot)-\tfrac{\cdot}{h}\mathcal Lh$ 展开的结果（[[wiki/concepts/infinitesimal-generator]]）。

## 在 DDBM 中的角色

[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] 用它把任意 VP/VE 扩散钉死终点 $x_T=y$，得到 forward bridge SDE（公式 5）：
$$
\mathrm dx_t=[f+g^2\,h(x_t,t,y,T)]\mathrm dt+g\,\mathrm dw_t,\quad h=\nabla_{x_t}\log p(x_T\mid x_t).
$$
- $h$ 项**解析可得**（高斯转移核），不用学；要学的只是反向桥的 score $s=\nabla\log q(x_t\mid x_T)$。
- 反向采样（公式 7）里出现的是 $\tfrac12 s-h$：注意 **$h$ 在 forward 中拉向 $y$，但在从 $y$ 到 $x_0$ 的反向采样里 $-h$（或 $-wh$）实际推动样本离开 $y$**（用户 p.5 批注）——这是 reverse bridge 不退化成"停在 $y$"的关键。

## 与其他概念的关系

- 推导引擎：[[wiki/concepts/infinitesimal-generator]]（生成元 / Kolmogorov backward）
- 应用：[[wiki/concepts/diffusion-bridge]]（用 h-transform 构造的桥）、[[wiki/methods/ddbm]]
- 对照：[[wiki/concepts/stochastic-interpolants|stochastic interpolants]] 提供**不用 Doob h-function** 构造桥的另一条路（Albergo & Vanden-Eijnden）；[[wiki/concepts/classifier-guidance|classifier guidance]] 同样是"在漂移上加一个 $\nabla\log(\cdot)$ 条件项"，但条件的是标签 $y$ 而非终点状态

## 出处与引用

- [[wiki/sources/zhouDenoisingDiffusionBridge2023]]（DDBM，公式 5）
- [[raw/notes/生成元方法对于SDE]]（用户推导）
- 经典：Doob；Särkkä & Solin 2019（Applied SDEs）
