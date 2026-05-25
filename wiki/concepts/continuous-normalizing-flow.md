---
type: concept
title: Continuous Normalizing Flow（CNF）
aliases: [Continuous Normalizing Flow, CNF, 连续归一化流]
tags: [flow-matching, ode, generative-model, normalizing-flow]
status: active
created: 2026-05-24
updated: 2026-05-24
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# Continuous Normalizing Flow（CNF）

> 概念页。CNF 是 [[wiki/concepts/flow-matching|Flow Matching]] 训练的对象；源：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]，原始提出 Chen et al. 2018（Neural ODE）。

## 一句话定义

用一个含时向量场 $v_t(x;\theta)$（神经网络）定义的**确定性流** $\phi_t$，把简单基分布连续地变形成复杂数据分布：
$$\frac{d}{dt}\phi_t(x)=v_t(\phi_t(x)),\qquad \phi_0(x)=x.$$

## 密度怎么走（push-forward + 换元）

$$p_t=[\phi_t]_*p_0,\qquad
p_t(x)=p_0(\phi_t^{-1}(x))\,\Big|\det\tfrac{\partial\phi_t^{-1}}{\partial x}(x)\Big|.$$
即"原密度 × 体积压缩系数"。等价地，沿轨迹的对数似然满足 instantaneous change of variables：
$$\frac{d}{dt}\log p_t(x_t)=-\nabla\!\cdot v_t(x_t)
\;\Rightarrow\;
\log p_1(x_1)=\log p_0(x_0)-\int_0^1\nabla\!\cdot v_t(x_t)\,dt.$$

## diffeomorphism（流的性质）

$\phi_t$ 是微分同胚：① 可逆；② 它与逆都光滑；③ 不同初始点不会在有限时间合并到同一点——保证密度变换良定义、可双向积分（→ 可逆编码 / inversion）。

## 与 score-based 的对照

CNF 把"关于 $p(x)$ 的全部知识"装进 $v(x,t)$；而 [[wiki/concepts/score-sde|score-based]] 把它装进 score $\nabla\log p$。[[wiki/concepts/probability-flow-ode|PF-ODE]] 其实就是"由训练好的 score 事后导出的一个 CNF"——这条线把 diffusion 与 CNF 接到了一起。

## 训练之痛 → FM 的解法

传统 CNF 训练（最大似然）要在前向/反传中**反复数值积分 ODE**（simulation-based），昂贵难 scale。[[wiki/concepts/flow-matching|Flow Matching]] 改为**回归一个预设路径的速度场**（simulation-free），绕开训练期解 ODE。

## 与其他概念的关系

- 训练法：[[wiki/concepts/flow-matching]] / [[wiki/concepts/conditional-flow-matching]]
- 采样近亲：[[wiki/concepts/probability-flow-ode]]（diffusion 事后导出的 CNF）
- 似然工具：与 [[wiki/concepts/fokker-planck-equation|连续性方程]] 同源（散度即密度变化率）

## 出处与引用

- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（用 CNF 作 FM 的载体）
- Chen et al. 2018（Neural ODE / CNF / instantaneous change of variables）
