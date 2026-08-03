---
type: concept
title: Transport Coupling
aliases: [coupling, transport plan, coupling rewiring, marginal preserving coupling, 耦合]
tags: [optimal-transport, flow-matching, rectified-flow]
status: active
created: 2026-05-26
updated: 2026-08-03
sources: ["[[wiki/sources/liuFlowStraightFast2022a]]"]
evidence: ["[[research/experiments/2026-08-02-reject-and-skip-toy-report]]", "[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]"]
---

# Transport Coupling

> 概念页。源：[[wiki/sources/liuFlowStraightFast2022a|Rectified Flow (Liu et al. 2022)]] §2 / §3。在 RF / FM / OT 三套语言里都是绕不开的对象，这一页给出本 wiki 的统一定义与最小记号。

## 定义

给定 $\mathbb R^d$ 上两分布 $\pi_0,\pi_1$，一个 **coupling**（或 **transport plan**）是一个 joint $(X_0,X_1)$，其边缘满足 $X_0\sim\pi_0$、$X_1\sim\pi_1$。所有 coupling 的集合记 $\Pi(\pi_0,\pi_1)$。

- **独立耦合 / product coupling**：$X_0\perp\!\!\!\perp X_1$，joint = $\pi_0\otimes\pi_1$。
- **确定性 coupling / Monge map**：存在 $T:\mathbb R^d\to\mathbb R^d$ 使 $X_1=T(X_0)$ 且 $T(X_0)\sim\pi_1$。
- **OT coupling**：最小化 $\mathbb E[c(X_1-X_0)]$（$c$ 凸）的 coupling，记 $\pi^\ast$。

## 为什么 coupling 是研究变量

- **同一对 $(\pi_0,\pi_1)$ 有无穷多 coupling**——选哪个会反过来决定生成轨迹的几何复杂度。
- 在 [[wiki/methods/rectified-flow|RF]] / [[wiki/concepts/flow-matching|FM]] 里，coupling 是训练目标 $\|v_\theta(X_t,t)-(X_1-X_0)\|^2$ 的**取期望对象**——换 coupling 等于换训练分布。
- 经典 diffusion 默认独立耦合（每个噪声 $X_T$ 与数据 $X_0$ 无对应关系）；这并非神圣，可换。

## Coupling Rewiring（来自 RF）

[[wiki/methods/rectified-flow|RF]] 的关键观察：用线性插值 $X_t=(1-t)X_0+t X_1$ 训出的 ODE 会**改写 coupling**。

记 RF 学到的最优速度场为 $v^\ast(x,t)=\mathbb E[X_1-X_0\mid X_t=x]$，沿 ODE 解
$$dZ_t=v^\ast(Z_t,t)\,dt,\quad Z_0\sim\pi_0$$
得到新 coupling $(Z_0,Z_1)$。两条关键性质：

**Marginal preserving（Thm 3.3）.** $\operatorname{Law}(Z_t)=\operatorname{Law}(X_t)$ 对一切 $t$；特别 $(Z_0,Z_1)\in\Pi(\pi_0,\pi_1)$。

**Convex transport cost non-increasing（Thm 3.5）.** 对一切凸 $c$，
$$\mathbb E[c(Z_1-Z_0)]\le \mathbb E[c(X_1-X_0)].$$

即：RF **保边缘、改 joint、传输代价单调不增**——这就是"coupling rewiring"。[[wiki/concepts/reflow|Reflow]] 是把它递推迭代。

## 直观：为什么会 rewire

- 线性插值轨迹**可以交叉**（依赖完整端点对）；
- ODE 在 $(x,t)$ 处速度唯一，**不能交叉**；
- 平方损失的最优解 = 条件期望，把交叉处多条轨迹"平均"成一条；
- 结果：原本 $(X_0^{(i)},X_1^{(i)})$ 的端点配对被打散重组成 $(Z_0,Z_1)$，但每端的边缘分布不变。

## 与 OT 的关系

- OT coupling $\pi^\ast$ 是 $\arg\min_{\Pi}\mathbb E[c(X_1-X_0)]$；
- RF 的单次 rewire 只保证**不变差**，不一定取到 $\pi^\ast$；
- [[wiki/concepts/reflow|reflow]] 迭代是否收敛到 $\pi^\ast$ —— 开放问题（Liu et al. 仅给单调性）。

## 在编辑场景的意涵（待验证）

- DDIM inversion 把数据 $x_0$ 编码到 $x_T$，等价于在 $(\pi_T,\pi_0)$ 上构造一个**确定性 coupling**。
- 编辑要求"改 $x_T$ 一点点 → $x_0$ 也变一点点"，这是对 coupling **连续性 / 局部性**的需求。
- RF 视角下，"DDIM inversion 失真"可重表述为"coupling rewiring 的失稳"——给出新的诊断与改进入口（待 thesis 验证）。

## 推理时 Coupling 干预

[[wiki/concepts/reject-and-skip]] 把 coupling 从训练阶段变量进一步扩展为**推理时离散决策变量**。它不要求重新训练 $v_\theta$，而是在检测到局部不可信速度后主动偏离 marginal ODE 的细步 coupling，跨越局部区域并在远端重新接入速度场。

二维解析实验中，这种偏离表现为：相对 RK4 的轨迹误差很大，但 endpoint outlier 降低且人工 conditional branch 基本保留。这说明“保边缘、改 joint”的 coupling 语言不仅能解释 [[wiki/concepts/reflow|reflow]]，也能解释采样器施加的结构化离散偏置。不过真实 CIFAR-10 1-RF 上尚未证明这一偏置改善生成分布，当前仍是待验证机制。

## 与其他概念的关系

- 框架：[[wiki/methods/rectified-flow]]、[[wiki/concepts/flow-matching]]
- 操作：[[wiki/concepts/reflow]]（递推 rewire）
- 推理时干预：[[wiki/concepts/reject-and-skip]]
- 路径：[[wiki/concepts/optimal-transport-path|OT 路径]]（coupling 给定后再选路径）

## 出处

- [[wiki/sources/liuFlowStraightFast2022a]] §2（transport mapping problem）与 §3.2–3.3（rewiring 与定理）
