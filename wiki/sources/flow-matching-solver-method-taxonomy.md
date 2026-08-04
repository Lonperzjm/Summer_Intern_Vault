---
type: source
title: Flow Matching Solver 方法分类
aliases: [Flow Matching solver taxonomy, FM 求解器分类, FM solver 方法分类]
tags: [flow-matching, solver, ode, sde, sampling, taxonomy]
status: active
created: 2026-08-04
updated: 2026-08-04
raw: "[[raw/notes/flow-matching solver 方法分类]]"
authors: []
venue: user-note
year: unknown
arxiv: unknown
---

# Flow Matching Solver 方法分类

> Source 页。原始资料是用户根据 `raw/notes/flowmatching_solver_methods_2026-07-29.xlsx` 整理的分类笔记，不是单篇论文。本文保存其比较框架与综合判断；未单独 ingest 原论文的方法、数值和性质仍需回到原文验证。

## Motivation

Flow Matching（FM）的确定性采样需要求解

$$
\frac{\mathrm d x}{\mathrm d t}=v_\theta(x,t).
$$

“更快的 solver”可能实际改动不同对象：积分公式、时间网格、历史复用、轻量辅助网络、轨迹选择，甚至单次网络前向的系统成本。若只比较步数或名义 NFE，会把数学精度、模型调用和墙钟加速混为一谈。

这份笔记的目标是建立统一口径，并把传统数值方法与 FM 专用采样方法分层放进同一张图中。

## Method

### 比较口径

至少同时记录五个量：

1. **步数**：积分区间被切成多少段；
2. **NFE**：昂贵速度网络实际被调用多少次；
3. **全局阶数**：在光滑性与小步长等条件成立时的累计误差阶；
4. **自适应性**：是否根据局部误差或当前样本动态接受、拒绝或调整一步；
5. **历史内存**：是否缓存先前状态或速度。

还应补报辅助网络前向、Jacobian/VJP、拒步、编译、通信和硬件条件。相同步数不意味着相同 NFE，相同 NFE 也不意味着相同墙钟或总 FLOPs。

### 传统数值方法

| 类别 | 代表 | 主要交换关系 | 在 FM 中的定位 |
|---|---|---|---|
| 显式单步 | Euler、Midpoint、Heun、Ralston、RK3、RK4 | 用当步多个 stage 换局部精度 | 固定步基线；Euler 是最低成本基线 |
| 嵌入式 RK | RK45、Tsit5 | 用高低阶解之差估计误差，可能拒步 | 高精度参考轨迹、离散误差与困难区诊断 |
| 线性多步 / 预测—校正 | Adams–Bashforth、ABM | 用历史缓存换热启动后较低的新 NFE | 历史复用型 FM solver 的经典参照 |
| 隐式刚性方法 | BDF、Radau | 用非线性求解和可能的 Jacobian 成本换稳定性 | 小模型参考与刚性诊断；大生成网络通常过贵 |
| 随机积分 | Euler–Maruyama、stochastic Heun | 显式处理扩散噪声 | 适用于 SDE，不可直接当作纯 FM ODE solver |

Euler 每步 1 NFE，局部截断误差为 $O(h^2)$、全局误差为 $O(h)$。但“$s$ 个 stage 就一定达到 $s$ 阶”不是一般规律；方法阶数取决于 Butcher 阶条件、光滑性与稳定性等假设，神经速度场在非光滑或困难区域也可能降阶。

### FM 专用方法：按改动对象分类

| 层 | 代表方法 | 改动对象 | 主要边界 |
|---|---|---|---|
| 模型专用 solver 参数优化 | Bespoke、BNS、Differentiable Solver Search | 时间点、更新系数、历史组合 | 需要校准；常绑定模型、CFG 与 NFE 预算 |
| 轻量辅助网络 | FlowTurbo、BA-Solver | 冻结 backbone，学习速度变化或区间内速度 | 不能只计 backbone NFE；需计训练与辅助前向 |
| Training-free 结构化积分 | Flow-Solver、RF-Solver、STORK、CAB | 插值、Taylor 结构、解析线性项、稳定性结构 | 高阶性依赖光滑性；部分推导绑定 RF 参数化 |
| 时间网格与预算 | BOSS、FastFlow、SharpEuler、Entropy Across the Bridge | NFE 放在哪些时间点、是否跳步 | 应固定基础积分器，单独消融网格收益 |
| 轨迹选择 / coupling 干预 | FDS、[[wiki/concepts/reject-and-skip|Reject-and-Skip]] | 改变进入基础 solver 的状态或离散 coupling | 目标不只是减小原 ODE 的离散误差 |
| 扩散 solver 适配 | A-FloPS、DPM-Solver++/UniPC 的流式适配 | 输出参数化与半线性结构转换 | 不能默认原生 FM 满足扩散侧推导条件 |
| 系统与并行加速 | SADA、ParaFlow | 单次前向成本或时间方向顺序深度 | 应报告 FLOPs、墙钟、吞吐、显存、通信与总调用量 |
| 基准与分析 | From Euler to Dormand–Prince | 统一比较而非提出新 solver | 小规模结论不能自动外推到大模型 |

这些层大多正交。例如时间网格可以搭配 Euler、Heun 或多步法；轨迹选择也可以叠加在某个基础 solver 之前。

## Results

该原始资料是方法分类与比较表，**没有原创实验、统一 benchmark 或可独立验收的新增指标**。它支持的是结构性结论，而不是“某方法普遍最优”的经验结论：

- solver 评测必须同时报告 NFE 与隐藏计算成本；
- 传统积分精度、预算分配、轨迹控制和系统加速属于不同层；
- 自适应高阶法适合作为参考解，但其动态 NFE 与拒步不利于固定延迟和批处理；
- 多步法热启动后可做到每步一次新评估，但要承担历史、变步系数与稳定域问题。

笔记中各专用方法的典型 NFE、训练成本和加速数字均视为**待原论文验证**；已独立 ingest 的 Bespoke、BNS、RF-Solver、FastFlow、FDS 与 BA-Solver，以各自 source 页为准。

## 关系

- [[wiki/concepts/ode-solver-taxonomy]]：承接本笔记的全景分类，并与已 ingest 原论文证据合并。
- [[wiki/concepts/flow-matching]]：明确“FM 可用 ODE solver”背后的成本与适用条件。
- [[wiki/methods/rectified-flow]]：区分 reflow 的训练期轨迹拉直、传统数值积分与 RF 专用 solver。
- [[wiki/concepts/probability-flow-ode]]：传统 ODE solver 也适用于 diffusion 的确定性 PF-ODE，但参数化转换须逐项核对。
- [[wiki/concepts/reject-and-skip]]：属于轨迹选择 / temporal coupling 干预，不属于传统高阶积分。

## 对我的 thesis 的启示

本次 ingest **不要求改写 working thesis**，但强化了当前研究方向的边界：Reject-and-Skip 的目标不是更准确复现 marginal ODE，而是在局部速度不可信时引入结构化离散偏置并改变 coupling。

因此实验应：

- 固定并明确基础 solver；
- 在严格等 NFE 下比较分布质量与失败尾部；
- 额外报告 detector、trial、rollback、出口搜索及拒绝步骤的真实成本；
- 同时记录 reference-trajectory error，但只把它解释为“偏离 marginal ODE 的程度”，不能把偏离本身当失败。

## 我的 takeaways

1. 步数、NFE、阶数、自适应性和历史内存必须分开报告。
2. 单步 RK、多步法与自适应 RK 交换的是不同资源，不能只按阶数排序。
3. FM 专用“solver”横跨积分、网格、辅助网络、轨迹控制和系统层，应按改动对象分层。
4. Reject-and-Skip/FDS 是轨迹选择或 coupling 干预，不是传统积分精度改进。
5. 该笔记是用户综合资料；未 ingest 方法的具体声明仍需原文验证。
