---
type: concept
title: Non-Markovian Diffusion（非马尔可夫前向过程族）
aliases: [non-Markovian diffusion, 非马尔可夫扩散, "DDIM 前向族"]
tags: [diffusion, sampling-acceleration]
status: active
created: 2026-05-14
updated: 2026-06-01
sources: ["[[wiki/sources/songDenoisingDiffusionImplicit2022]]", "[[wiki/sources/zhengDiffusionBridgeImplicit2025]]"]
---

# Non-Markovian Diffusion（非马尔可夫前向过程族）

## 一句话定义

把 [[wiki/concepts/diffusion-process|DDPM 的前向过程]]从"逐步马尔可夫加噪"松绑成**一族以 $x_0$ 为条件的非马尔可夫过程**，只要求它们的**边缘分布** $q(x_t\mid x_0)$ 与 DDPM 相同；由此整族过程共享同一训练目标，但反向链可以更短、可确定。

## 数学/技术细节

DDPM 的训练目标 $L_\text{simple}$ 只用到边缘 $q(x_t\mid x_0)=\mathcal N(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I)$，**从不要求前向是马尔可夫链**。DDIM 据此定义一族由 $\sigma\in\mathbb R^T_{\ge0}$ 索引的过程 $q_\sigma$，其反向转移闭式为

$$
q_\sigma(x_{t-1}\mid x_t,x_0)=\mathcal N\!\left(\sqrt{\bar\alpha_{t-1}}x_0+\sqrt{1-\bar\alpha_{t-1}-\sigma_t^2}\cdot\frac{x_t-\sqrt{\bar\alpha_t}x_0}{\sqrt{1-\bar\alpha_t}},\ \sigma_t^2 I\right)
$$

均值函数被刻意设计成使边缘对所有 $t$ 都等于 DDPM 的边缘。关键结论（DDIM Theorem 1）：对所有 $\sigma>0$，变分目标 $J_\sigma=L_\gamma+C$ —— 整族共享目标。

- $\sigma_t=\sqrt{\tilde\beta_t}$ → 过程退回马尔可夫的 DDPM。
- $\sigma_t=0$ → 过程确定性（DDIM）。
- $\sigma$ 介于两者之间 → 一条随机性可调的连续谱。

因为不依赖马尔可夫性，反向链可只定义在时间步的**任意子序列**上 → 跳步加速采样。

### Bridge 版（DBIM）：同一招用在扩散桥上

[[wiki/sources/zhengDiffusionBridgeImplicit2025|DBIM]] 把这套"保边缘、松绑路径"原样搬到 [[wiki/methods/ddbm|DDBM]] 的扩散桥上——**结构完全对位 DDIM:DDPM ＝ [[wiki/methods/dbim|DBIM]]:DDBM**：

| | DDIM（扩散） | DBIM（桥） |
|---|---|---|
| 保住的边缘 | $q(x_t\mid x_0)$ | $q(x_{t_n}\mid x_T)$（Prop 3.1） |
| 系数 | $\alpha_t,\sigma_t$ | 桥的 $a_t,b_t,c_t$ |
| 随机旋钮 | $\sigma_t$（$\eta$） | $\rho_n$ |
| 确定极限 | $\sigma{=}0$ → DDIM ODE | $\rho{=}0$ → 桥 ODE |
| 复用网络 | ε 网络 | DDBM bridge score $s_\theta$ |

差异：桥在**初始步 $c_T{=}0$ 处奇异**，DBIM 用 **booting noise**（固定 $x_T$ 下的 latent）补上，从而 $\rho{=}0$ 仍能 faithful encoding/reconstruction。

## 与其他概念的关系

- 是对 [[wiki/concepts/diffusion-process]] 中"前向为固定马尔可夫链"假设的松绑——保留边缘、丢掉路径。
- 之所以能复用网络，根因在 [[wiki/concepts/epsilon-parameterization]]：ε 目标只看边缘。
- $J_\sigma$ 是 [[wiki/concepts/variational-bound-elbo]] 的非马尔可夫推广。
- 取连续时间极限即 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]。

## 在 text-guided editing 中的作用

- 把"前向过程"从一条死链变成一个**可设计的对象**：编辑方法可以借此选择在哪条轨迹、哪些时间步上注入条件。
- $\sigma=0$ 的确定性是 DDIM inversion 的前提——inversion-based 编辑的整条技术路线由此打开。

## 出处与引用

- [[wiki/sources/songDenoisingDiffusionImplicit2022]]（Song et al., DDIM, ICLR 2021 —— 非马尔可夫族与 Theorem 1）
