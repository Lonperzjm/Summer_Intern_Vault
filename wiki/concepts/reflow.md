---
type: concept
title: Reflow / Rectification
aliases: [reflow, rectification, k-rectification, 直线化, 轨迹拉直]
tags: [flow-matching, rectified-flow, sampling-acceleration, ode]
status: active
created: 2026-05-26
updated: 2026-05-26
sources: ["[[wiki/sources/liuFlowStraightFast2022a]]"]
---

# Reflow / Rectification

> 概念页。源：[[wiki/sources/liuFlowStraightFast2022a|Liu, Gong & Liu 2022 (Rectified Flow)]]。Reflow 是 [[wiki/methods/rectified-flow|Rectified Flow]] 区别于 [[wiki/concepts/flow-matching|FM]] 的最重要构成性步骤。

## 一句话定义

把一条已训好的 ODE 所诱导的新 coupling $(Z_0,Z_1)$ 当作**新输入**，再用 [[wiki/methods/rectified-flow|RF]] 训一条新 ODE——递推下去，**直线度单调增、凸传输代价单调降**，极限时轨迹是常速直线，单步 Euler 即精确。

## 形式化

把 RF 训练 + ODE 求解视作算子
$$\mathrm{Rect}:(X_0,X_1)\mapsto(Z_0,Z_1=\Phi_1(Z_0)).$$
递推
$$(X_0^{k+1},X_1^{k+1})=\mathrm{Rect}\!\left((X_0^k,X_1^k)\right),\quad k=0,1,2,\dots$$
得到 1-rectified、2-rectified、…、k-rectified flow。

## Straightness 测度

$$S(Z)=\int_0^1\mathbb E\!\left[\,\big\|(Z_1-Z_0)-\dot Z_t\big\|^2\,\right]dt.$$
- $S(Z)=0$ ⟺ 轨迹处处常速直线（$\dot Z_t \equiv Z_1-Z_0$）；
- Liu et al. 证 $S(Z^{k+1})\le S(Z^k)$（单调不增）；
- 实测中 $k=1$ 已带来量级提升，$k=2,3$ 收益递减。

## 凸代价非增（Thm 3.5）

对任意凸 $c:\mathbb R^d\to\mathbb R$，
$$\mathbb E[c(Z_1^{k+1}-Z_0^{k+1})]\le \mathbb E[c(Z_1^k-Z_0^k)].$$
推论：$L^2$ 传输代价（OT 经典目标）随 $k$ 单调降。⚠️ **不**断言极限达到 Monge OT；只保证不变差。

## 为什么这是大事

| 旧观点 | reflow 带来的修正 |
|---|---|
| 加速 = 改采样器（DDIM 跳步、PC corrector、高阶 ODE solver） | 加速也可来自**改训练阶段轨迹本身** |
| 路径是 SDE 设计的副产物 | 路径是可被**迭代改造**的对象 |
| 1-step 生成要靠蒸馏 | 蒸馏 = 在已经接近直线的 reflow 终态上做"最后一脚"，代价小得多 |

这把 [[wiki/overview]] 推论 3 "采样加速可来自路径/轨迹设计" 推到极限：**训练阶段直接把 ODE 改成直线**。

## 与蒸馏的分工

- **Reflow**：迭代式**重训**整条 ODE，目标"轨迹变直"，输出仍是 ODE；
- **Distillation**：在 reflow 后做 amortized one-shot 回归 $\hat T(Z_0)\approx Z_1$，输出是一个**生成器函数**（不再是 ODE）。
- 两者**串联**而非互斥：先 reflow→ 再 distill 是 RF 一系实现 1-step 生成的标准管线。

## 与其他概念的关系

- 框架：[[wiki/methods/rectified-flow]]、[[wiki/concepts/flow-matching]]、[[wiki/concepts/optimal-transport-path]]
- coupling 视角：[[wiki/concepts/transport-coupling]]（reflow 改的就是 coupling 的 joint）
- 采样近亲对照：[[wiki/concepts/probability-flow-ode|PF-ODE]] / [[wiki/methods/ddim]]（这些走"动采样器"，reflow 走"动训练"）

## 开放问题

- Reflow 在何种条件下收敛到 Monge OT？目前只有"单调改善"。
- 极限直线后是否丢失多样性（mode collapse 类风险）？经验上 $k$ 不宜过大。
- 在条件生成 / 编辑场景，reflow 后的 RF 是否仍能保证 inversion 的往返闭合？开放。

## 出处

- [[wiki/sources/liuFlowStraightFast2022a]] §3.3（reflow 流程 + 定理）
