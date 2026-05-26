---
type: source
title: "Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow"
aliases: [Rectified Flow 原文, "Liu et al. 2022", "Liu, Gong & Liu 2022", rectified-flow-paper]
tags: [flow-matching, rectified-flow, ode, transport, sampling-acceleration, foundational]
status: stable
created: 2026-05-26
updated: 2026-05-26
raw: "[[raw/literature-notes/liuFlowStraightFast2022a]]"
authors: [Xingchao Liu, Chengyue Gong, Qiang Liu]
venue: arXiv 2209.03003 / ICLR 2023
year: 2022
arxiv: "2209.03003"
---

# Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow

> 文献笔记：[[raw/literature-notes/liuFlowStraightFast2022a]] · arXiv [2209.03003](http://arxiv.org/abs/2209.03003) · Liu, Gong & Liu 2022（ICLR 2023）
> 与 [[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023 FM]]、Albergo & Vanden-Eijnden 2022（Stochastic Interpolants）**同时期并行**：都把生成建模重塑为「直接学一条 ODE/path 的速度场」。RF 的差异化贡献是**线性插值 + 迭代 rectification（reflow）+ 任意 coupling 接口**。
> ⚠️ 时间约定与 [[wiki/concepts/flow-matching|FM]] 同：$t=0$ 是 $\pi_0$（源 / 噪声），$t=1$ 是 $\pi_1$（目标 / 数据），生成沿 $t:0\to 1$ 正向积分。与 [[wiki/methods/ddpm|DDPM]]/[[wiki/concepts/score-sde|Score SDE]] 的 $t:0\to T$（数据→噪声）方向**相反**。

## 一句话

给任意两分布 $(\pi_0,\pi_1)$ 的任意 coupling $(X_0,X_1)$，学一条 ODE 让它**尽量走直线**地把 $\pi_0$ 输运到 $\pi_1$；训练就是对线性插值 $X_t=(1-t)X_0+t X_1$ 做最小二乘回归 $v_\theta(X_t,t)\approx X_1-X_0$。把训练好的 ODE 诱导的 $(Z_0,Z_1)$ 作为新 coupling **再训一遍**（[[wiki/concepts/reflow|reflow / rectification]]），凸传输代价非增、直线度单调增——极限直线时**单步 Euler 即精确**。同一框架同时覆盖生成、image-to-image 与域适应。

## Motivation

作者把"分布间运输"作为统一问题：给定 $X_0\sim\pi_0$、$X_1\sim\pi_1$，找 transport map $T$ 使 $T(X_0)\sim\pi_1$。已有路线各有局限：

- GAN：minimax 训练不稳；
- MLE / normalizing flow：需要可逆架构 + log-det，表达力受限；
- diffusion / [[wiki/concepts/score-matching|score matching]]：路径设计绑定 SDE，采样要数十~数百步；
- 经典 OT：高维数值求解贵。

诉求：**避开 GAN 的 minimax、MLE 的可逆、diffusion 的步数**，写成一个**普通的 L2 回归**问题，且能 scale。

## Method

### 1. Rectified Flow 主算法

给 coupling $(X_0,X_1)$，定义线性插值
$$X_t=(1-t)X_0+t X_1,\qquad t\in[0,1].$$
学速度场 $v_\theta(\cdot,t)$ 最小化
$$\mathcal L(\theta)=\int_0^1\mathbb E_{(X_0,X_1)}\!\left[\,\|v_\theta(X_t,t)-(X_1-X_0)\|^2\,\right]dt.$$
平方损失下最优解是 **marginal velocity / conditional expectation**
$$v^\ast(x,t)=\mathbb E[\,X_1-X_0\mid X_t=x\,].\tag{1}$$
采样时用 ODE 正向积分
$$dZ_t=v_\theta(Z_t,t)\,dt,\qquad Z_0\sim\pi_0,\ t:0\to 1.$$

> **形式上同构于 FM-OT 路径**（[[wiki/concepts/optimal-transport-path]]）取 $\sigma_{\min}=0$。差异**不在公式本身**，而在**接口与流程**：RF 把"任意 coupling"作为一等公民输入，并把 reflow 作为构成性步骤。详见 §"与 FM/OT 路径的关系"。

### 2. Rewiring：非因果插值 → 因果 ODE

$X_t$ 是**非因果**的（依赖完整端点对 $(X_0,X_1)$），其轨迹**允许交叉**。ODE 在 $(x,t)$ 处速度唯一，**不能交叉**。式 (1) 的"条件期望"恰恰把交叉处的多条轨迹**平均**成一条——结果是 ODE 解 $(Z_0,Z_1)$ 构成**新的 deterministic coupling**：
$$Z_1=\Phi_1(Z_0),\quad Z_0\sim\pi_0.$$

![[liuFlowStraightFast2022a-1779768003417.webp]]

> (a) 原始 coupling 的线性插值显式交叉 → (b) RF 把交叉重 wire 成不交叉的弯轨迹 → (c)(d) 用新 coupling 再做一次 RF，轨迹接近直线。

### 3. 两条核心定理

**Theorem 3.3（Marginal preserving）.**
$$\operatorname{Law}(Z_t)=\operatorname{Law}(X_t),\quad \forall t\in[0,1].$$
特别地 $Z_0\sim\pi_0$、$Z_1\sim\pi_1$——新 coupling 的边缘与原 coupling 相同，只是 joint 不同（这正是 [[wiki/concepts/transport-coupling|coupling rewiring]]）。

**Theorem 3.5（Convex transport cost non-increasing）.** 对任意凸 $c:\mathbb R^d\to\mathbb R$，
$$\mathbb E[c(Z_1-Z_0)]\le \mathbb E[c(X_1-X_0)].$$
即新 coupling **不会比原 coupling 在任何凸代价下更差**。注意：**不**断言达到 Monge OT；只断言单调改善。

### 4. Reflow（k-rectification）

把 RF 视作算子 $\mathrm{Rect}$，从任意初始 coupling $(X_0^0,X_1^0)$ 出发递推
$$(X_0^{k+1},X_1^{k+1})=\mathrm{Rect}\!\left((X_0^k,X_1^k)\right).$$
作者证：**直线度** 
$$S(Z)=\int_0^1 \mathbb E\!\left[\,\|(Z_1-Z_0)-\dot Z_t\|^2\,\right]dt$$
随 $k$ 单调降，趋于 0 时轨迹是常速直线；此时
$$Z_t=(1-t)Z_0+t Z_1,\quad v(Z_t,t)=Z_1-Z_0,$$
**单步 Euler** $Z_1=Z_0+v_\theta(Z_0,0)$ **即精确**。详见 [[wiki/concepts/reflow]]。

### 5. Distillation

Reflow 把轨迹拉直后，进一步可以**蒸馏**：学一个 amortized map $\hat T_\theta(Z_0)\approx Z_1$ 直接回归，得到**真正的 1-step 生成器**。这与扩散蒸馏（progressive distillation / consistency distillation）路线在目的上重合，但 RF 的"原料"是已经接近直线的轨迹——蒸馏代价显著更小。

### 6. Velocity field 的具体形式（toy / 分析）

当 $\pi_0,\pi_1$ 解析可写时，
$$v_X(z,t)=\mathbb E\!\left[\frac{X_1-z}{1-t}\,\eta_t(X_1,z)\right],\quad
\eta_t(X_1,z)=\frac{\rho(\tfrac{z-tX_1}{1-t}\mid X_1)}{\mathbb E[\rho(\tfrac{z-tX_1}{1-t}\mid X_1)]}.\tag{4}$$
其中 $\rho(x_0\mid x_1)$ 是 coupling 的条件密度。若 $\pi_0$ 是 dirac/离散（端点对完全 overfit），$v_X$ 退化。**修复**：给 $X_0$ 加 $\xi\sim\mathcal N(0,\sigma^2 I)$ 得到 $\tilde X_0=X_0+\xi$ 作为 smoother，使 $\rho(x_0\mid x_1)$ 良定。

🟢 用户标注：与 [[wiki/methods/rectified-flow|RF]] 后续 / FlowCycle 的 smoother 设计同思路；🔴 用户标注："不加 smoother 完全过拟合" 与 FlowCycle 的现象一致——这是论文给出的"为什么必须加噪"的根因，不只是工程 trick。

### 7. Nonlinear extension

把直线插值换成任意 $X_t=\phi_t(X_0,X_1)$（满足端点条件）即得 nonlinear rectified flow——**与 FM 的 conditional path 框架等价**。用户标注："就是 FM 的方法"——确为同时期独立提出、形式互通。

## Results

| 任务 | 数据集 | 关键指标 | 结论 |
|---|---|---|---|
| 无条件生成 | [[wiki/benchmarks/cifar10\|CIFAR-10]] | FID（1-step） | 一次 reflow 即把 1-step FID 从 ~378（直 1-step EM）压到个位 |
| 无条件生成 | CIFAR-10 | FID（full NFE） | 与 DDPM/Score SDE 同档 |
| Image-to-image | cat ↔ dog（AFHQ）等 | 视觉质量 | 用同一框架统一处理（$\pi_0,\pi_1$ 均为图像） |
| 域适应 | toy + 标准 benchmark | 准确率 | 把 RF 的 transport 用作 feature alignment |

> 具体 FID 数字按"FM-OT 在同架构下三项一致最好"的 [[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman 2023]] 报告交叉验证：RF 与 FM-OT 在 NFE-质量曲线上接近，**RF 的独特卖点是 reflow 把曲线左端（few-step / 1-step）显著抬高**。

## 关系

### 与 [[wiki/concepts/flow-matching|Flow Matching]] / [[wiki/concepts/optimal-transport-path|OT 路径]]

- **公式同构**：RF 的线性插值 = FM-OT 路径在 $\sigma_{\min}=0$。
- **不同**：
  - RF 把 $(\pi_0,\pi_1)$ 写成**任意两分布**且 coupling 可任取；FM 默认 $\pi_0=\mathcal N(0,I)$、coupling 为独立耦合。
  - RF 把 **reflow** 作为构成性步骤；FM 不包含 reflow。
  - RF 的目标变量是 $X_1-X_0$（端点差）；FM 的目标变量是路径切向 $u_t(x\mid x_1)$，对 OT 路径恰好也是端点差形式。
- 实操上 SD3 / FLUX 训的就是"FM 框架 + RF 路径 + (可选) reflow"，名字按厂商口味叫 rectified flow 或 flow matching。

### 与 [[wiki/concepts/probability-flow-ode|PF-ODE]] / [[wiki/methods/ddim|DDIM]]

- 三者都是**确定性 ODE 生成**。
- PF-ODE / DDIM：score 网络 → 事后导出 ODE，训练是 ε/score 监督。
- RF：**直接训速度场**，且通过 reflow 把轨迹拉直，把"few-step 采样"从**采样器问题**降为**训练阶段问题**——是 overview 推论 3"加速可来自路径设计"的极限版本。

### 与 [[wiki/methods/rectified-flow]]（本 wiki 的方法页）

method 页是 RF 方法族的常态入口（含下游 SD3/FLUX、RF-Inversion 等编辑工作的挂钩点）。本 source 页只承担"忠实复述原文"。

### 与 [[wiki/concepts/score-matching|DSM]] 的同构

L2 回归 + "monolithic 边缘场 = 条件期望"的结构与 [[wiki/concepts/conditional-flow-matching|CFM]]/[[wiki/concepts/score-matching|DSM]] 同源——RF 是这套"用条件目标回归边缘目标"思想的第三个独立化身（DSM/CFM/RF），梯度等价性的证明几乎是同一模板。

## 对我的 thesis 的启示

- 🟣 **加速可设计在训练阶段，而不只是采样器**：reflow 把 [[wiki/overview]] 推论 3 的"采样链可改"推到极限——**轨迹本身可被改造**。本 thesis 关心的编辑场景反复采样，若 backbone 已是 RF 一族（SD3 / FLUX），inversion / guidance 的设计能否利用"接近直线"的轨迹来稳定 fidelity↔controllability？这是一个具体可做的研究杠杆。
- 🟣 **coupling 是被忽视的研究变量**：RF 显式承认 coupling 可任取并可被 rewire。在编辑场景里，inversion 出的 $(x_0^{\text{src}}, x_T)$ 也是一个 coupling——RF 视角能否解释/改进 DDIM inversion 的 "rewiring 失败"现象？与你已有的 FlowCycle 工作直接接合。
- 🟣 **smoother / $\rho(x_0\mid x_1)$ 良定性**（用户 p.8 批注）：RF 给出的 smoother 修复（$\tilde X_0 = X_0+\xi$）从原理上回答了"为什么 FlowCycle 必须加噪"——这是定理 (4) 的退化条件，不是工程 hack。该结论可写进 thesis 的"方法稳定性"一章。
- 🔴 **凸代价非增 ≠ 收敛到 OT**：Thm 3.5 只断言单调改善。reflow 收敛到 OT 的条件、收敛速率仍是开问题——若 thesis 想从"为什么直线化能改善编辑"出发做 claim，需要小心这一点。

## 待调研方向（来自 🔵 与开放问题）

- [ ] Reflow 收敛性：在什么条件下 $k\to\infty$ 收敛到 OT map？已有数学进展？
- [ ] RF + 编辑：RF-Inversion（Wang et al. 2024）—— RF backbone 上的 inversion；Esser et al. 2024（SD3）—— RF + transformer 的工业实现。
- [ ] Consistency Models（Yang Song）与 reflow 的目的重叠：都想 1-step；两者能否互补？
- [ ] Stochastic Interpolants（Albergo & Vanden-Eijnden 2022）：第三个并行工作，给出 SDE/ODE 统一的 interpolant 框架——值得 ingest 看与 RF/FM 的统一视角。

## 出处与引用

- arXiv [2209.03003](http://arxiv.org/abs/2209.03003)；ICLR 2023
- 同时期并行：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023 (FM)]]、Albergo & Vanden-Eijnden 2022（Stochastic Interpolants，待 ingest）
- 直接下游：SD3（Esser et al. 2024，待 ingest）、FLUX（Black Forest Labs，待 ingest）、RF-Inversion 系（待 ingest）
- 作者实体：[[wiki/entities/xingchao-liu]]、[[wiki/entities/qiang-liu]]
