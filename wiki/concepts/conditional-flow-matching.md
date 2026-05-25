---
type: concept
title: Conditional Flow Matching（CFM）
aliases: [Conditional Flow Matching, CFM, 条件流匹配]
tags: [flow-matching, training-objective, generative-model]
status: active
created: 2026-05-24
updated: 2026-05-24
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# Conditional Flow Matching（CFM）

> 概念主页。源：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]。是 [[wiki/concepts/flow-matching|Flow Matching]] 可训练化的核心技巧。

## 一句话定义

边缘向量场 $u_t(x)$ intractable，无法直接回归；CFM 改用**每个数据点 $x_1$ 的条件向量场 $u_t(x\mid x_1)$**（有闭式）作监督，并证明其梯度与不可行的边缘目标**完全相同**。

## 边缘化构造

$$p_t(x)=\int p_t(x\mid x_1)\,q(x_1)\,dx_1,\qquad
u_t(x)=\int u_t(x\mid x_1)\,\frac{p_t(x\mid x_1)\,q(x_1)}{p_t(x)}\,dx_1.$$
（类比流体 $\bar v=\overline{\rho v}/\bar\rho$；用连续性方程可证"边缘场生成边缘路径"，即 Theorem 1。）

## 目标与梯度等价（Theorem 2）

$$\mathcal L_{\mathrm{CFM}}(\theta)=\mathbb E_{t,\,q(x_1),\,p_t(x\mid x_1)}\big\|v_t(x;\theta)-u_t(x\mid x_1)\big\|^2.$$
当 $p_t(x)>0$，$\mathcal L_{\mathrm{CFM}}$ 与 $\mathcal L_{\mathrm{FM}}$ 至多差一个与 $\theta$ 无关的常数，故
$$\nabla_\theta\mathcal L_{\mathrm{FM}}=\nabla_\theta\mathcal L_{\mathrm{CFM}}.$$

## 证明骨架（= DSM 同一套路）

把 L2 对 $x$ 展开，常数项剔除后只剩交叉项，其中
$$\mathbb E_{x_1\mid x,t}\big[u_t(x\mid x_1)\big]=u_t(x),$$
即**最优网络收敛到条件期望 = 边缘场**。这与 [[wiki/concepts/score-matching|denoising score matching]]「用条件 score 当监督却学到边缘 score」是**同构**的论证——论文 §5 亦明言 CFM "draws inspiration from" Vincent 2011，只是把回归对象从 score 换成 vector field。

> 一句话记忆：**CFM : FM ＝ DSM : (intractable) score matching**。

## 为什么关键

没有 CFM，FM 只是漂亮但不可训的定义；有了它，训练就退化为"采 $x_1$、采噪声、采 $t$ → 构造 $x_t$ → 回归闭式条件场"，simulation-free 且可 scale 到高维。

## 与其他概念的关系

- 上层：[[wiki/concepts/flow-matching]]（CFM 是其训练实现）
- 同构对照：[[wiki/concepts/score-matching]]（条件→边缘恒等式）
- 闭式条件场来自高斯条件路径 → [[wiki/concepts/optimal-transport-path]] 等具体设计

## 出处与引用

- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（Theorem 1 & 2）
- Vincent 2011（DSM，CFM 的灵感来源）
