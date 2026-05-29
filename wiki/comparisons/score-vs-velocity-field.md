---
type: comparison
title: "Score ∇log p vs Velocity Field v：保守场 vs 一般场"
aliases: [score vs velocity field, 保守场 vs 非保守场, "∇log p vs v_t", score-velocity-equivalence]
tags: [diffusion, flow-matching, score-based, ode, sde, conceptual]
status: stable
created: 2026-05-29
updated: 2026-05-29
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# Score $\nabla\log p$ vs Velocity Field $v$：保守场 vs 一般场

> 起因：query「SDE 和 Flow Matching 的 ODE 是并列的吗」。结论是"并列"对一半——要分**结果层**（可互转）与**语义层**（对象本质不同）两层看。本页把这个区分定形，供 thesis 引用。

## 一句话

[[wiki/concepts/score-matching|score]] $\nabla_x\log p_t$ 按定义就是标量场的梯度 → **保守场（无旋）**；[[wiki/concepts/flow-matching|Flow Matching]] 的速度场 $v_t(x)$ 解连续性方程、只受散度约束 → **一般场（可有旋），写不成 $\nabla(\cdot)$**。diffusion 把生成动力学限制在"保守速度场"子集，FM 放到"任意满足边缘约束的速度场"大空间——这是 FM 更一般、且 [[wiki/concepts/optimal-transport-path|OT 路径]]能"走直线"的数学根源。

## 三个对象，先别混成两个

| 对象 | 框架 | 随机/确定 | 核心量 |
|---|---|---|---|
| reverse SDE | [[wiki/concepts/score-sde\|Score SDE]] | 随机 | score $\nabla\log p_t$ |
| [[wiki/concepts/probability-flow-ode\|PF-ODE]] | Score SDE | 确定 | score（事后导出 ODE） |
| FM 的 ODE | [[wiki/concepts/flow-matching\|Flow Matching]] | 确定 | velocity field $v_t$（直接训练） |

- 框架**内部**：reverse SDE 与 PF-ODE 不是并列，是同一族 $\{p_t\}$ 的随机/确定两种采样。
- 真正同类可比的是 **PF-ODE 的速度场** vs **FM 的速度场**——都是确定性 ODE 的速度场，差别在"事后从 score 导出" vs "直接回归训练"。

## 语义层：为什么 $v$ 一般写不成 $\nabla\log p$

### Score 必是保守场

$$
\text{score}(x,t) := \nabla_x\log p_t(x)
$$
是标量函数 $\log p_t$ 的梯度，故**旋度恒为零**（单连通域上 $\nabla\times\nabla\phi=0$）。这是定义层面的必然——它"长成梯度的样子"不是巧合。

### FM 的速度场只受散度约束

$v_t$ 由连续性方程定义：
$$
\partial_t p_t(x) + \nabla_x\!\cdot\big(p_t(x)\,v_t(x)\big) = 0.
$$
该方程**只约束 $p_t v_t$ 的散度**，对旋度无要求。后果：

1. **$v_t$ 不唯一**：给定 $p_t$，任意叠加一个无散度场 $u$（$\nabla\!\cdot(p_t u)=0$）仍生成同一条 path。FM 通过选定条件路径 + 取条件期望（[[wiki/concepts/conditional-flow-matching|CFM]]）从这个无穷解集里**选出一个**特定的 $v_t$。
2. **被选出的 $v_t$ 一般非保守**：[[wiki/concepts/optimal-transport-path|OT 直线路径]]的边缘速度场一般有旋度，**不存在标量 $\phi$ 使 $v_t=\nabla\phi$**——所以写不成 $\nabla\log(\cdot)$ 那种形式。

### 硬事实：diffusion 的 PF-ODE 速度场恰好是保守的

PF-ODE 速度场 $\tilde f = f - \tfrac12 g^2\nabla\log p_t$：
- **VE**（$f=0$）：$\tilde f \propto \nabla\log p_t$ —— 纯梯度场；
- **VP**（$f=-\tfrac12\beta x$）：$f$ 项 $=\nabla(-\tfrac14\beta\|x\|^2)$ 也是梯度 → $\tilde f$ 仍是**保守场**。

即：**diffusion 导出的确定性流，速度场永远落在保守场子集**。FM 不受此限——这正是它"能走直线、能更一般"的来源（score 流要绕弯，因为它被钉在 $\nabla\log p_t$ 的形状上）。

## 结果层：什么时候"可互转"成立

"$v \leftrightarrow \nabla\log p$ 线性互转"**只在固定高斯路径下成立**，前提是两者描述**同一条** probability path $\{p_t\}$：

- 给定同一族 $p_t$：score 唯一；生成它的 $v_t$ 不唯一（差一个无散度场）。
- **高斯路径（VP/VE）**：FM 选出的典范速度场恰好 = PF-ODE 速度场，于是有闭式线性关系 $v_t \leftrightarrow \nabla\log p_t$。此时 SDE / PF-ODE / FM 三者在该 path 上**等价**。
- **换成 OT 直线路径**：$p_t$ 这族分布本身就变了，速度场成非保守场——**没有一个 score 能等价表达它**（仍可定义 $\nabla\log p_t$，但它生成的是另一条弯曲 path，不是那条直线）。

> 易错点：把"高斯路径下可互转"误推广成"score 和 velocity 永远等价"。一旦路径非高斯 / 速度场非保守，互转就断了。

## 收口（两层说法）

- **结果层（仅高斯路径）**：$v$ 与 $\nabla\log p$ 线性互转 → 同一条 diffusion path 上 SDE / PF-ODE / FM 等价。
- **语义层（一般情形）**：score 必是保守场、$v$ 一般非保守 → FM 的 $v$ **不能**写成 $\nabla\log p$；diffusion 是 FM 在"保守速度场 + 高斯路径"特例上的退化。

## 对 thesis 的意义

- 解释了为什么 [[wiki/methods/rectified-flow|Rectified Flow]] / FM 能把轨迹**拉直**而 score 流不能——直线速度场是非保守场，超出 score 能表达的范围（[[wiki/concepts/reflow|reflow]]、[[wiki/concepts/optimal-transport-path|OT 路径]]）。
- 给 [[wiki/overview]] 推论 1「真正的不变量是范式、组件落在可变性光谱上」一个更锋利的刻画：从 score 到 velocity field 是**把生成动力学从保守场子集解放到一般场**——这是"训练目标"那一档可演化性的数学本质，不只是"换个回归目标"。
- caveat 衔接：推论 2 讨论"在哪个 $t$ 介入"时，diffusion 的 $t$（数据→噪声）与 FM 的 $t$（噪声→数据）方向相反（见 [[wiki/sources/lipmanFlowMatchingGenerative2023|FM]] 时间约定 caveat），与本页"对象不同"是两个独立的跨范式 caveat，勿混。

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（Score SDE：reverse SDE / PF-ODE / score 是核心量）
- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（Flow Matching：velocity field、连续性方程、Gaussian 路径把 diffusion 收为特例）
- 相关概念：[[wiki/concepts/probability-flow-ode]]、[[wiki/concepts/flow-matching]]、[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/optimal-transport-path]]、[[wiki/concepts/score-matching]]、[[wiki/concepts/fokker-planck-equation]]
