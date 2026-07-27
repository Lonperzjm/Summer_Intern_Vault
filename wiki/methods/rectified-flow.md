---
type: method
title: Rectified Flow
aliases: [Rectified Flow, rectified flow, RF, "Liu et al. 2022"]
tags: [flow-matching, ode, rectified-flow, sampling-acceleration]
status: active
created: 2026-05-24
updated: 2026-07-27
sources: ["[[wiki/sources/liuFlowStraightFast2022a]]", "[[wiki/sources/lipmanFlowMatchingGenerative2023]]", "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]", "[[wiki/sources/labsFLUX1KontextFlow2025]]", "[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]"]
family: flow-matching
---

# Rectified Flow（RF）

> 方法主页。原文 [[wiki/sources/liuFlowStraightFast2022a|Liu, Gong & Liu 2022 (ICLR 2023)]] 已 ingest。本页负责 RF 在 vault 中的"方法族入口"：把 RF 与 [[wiki/concepts/flow-matching|FM]]、[[wiki/concepts/optimal-transport-path|OT 路径]]、[[wiki/concepts/probability-flow-ode|PF-ODE]] 的精确关系厘清，并承接其工业下游（SD3 / FLUX / RF-Inversion）。

## 一句话

学一条把 $\pi_0$（噪声）尽量**直线**输运到 $\pi_1$（数据）的 ODE，训练就是对线性插值做 L2 回归；用 [[wiki/concepts/reflow|reflow]] 迭代把轨迹拉直——极限直线时**单步 Euler 即精确**。

## 核心算法

1. 给 coupling $(X_0,X_1)$，$X_t=(1-t)X_0+t X_1$；
2. 训 $v_\theta(X_t,t)\approx X_1-X_0$（L2）；最优解 $v^\ast(x,t)=\mathbb E[X_1-X_0\mid X_t=x]$；
3. 采样 ODE $dZ_t=v_\theta(Z_t,t)\,dt$, $t:0\to 1$；
4. 用 $(Z_0,Z_1)$ 作新 coupling 再训（[[wiki/concepts/reflow|reflow]]），递推 $k$ 次；
5. 可选：在 reflow 后蒸馏出 amortized 1-step 生成器。

## 与 [[wiki/concepts/flow-matching|FM]] / [[wiki/concepts/optimal-transport-path|OT 路径]] 的精确异同

| 维度 | Rectified Flow（Liu 2022） | Flow Matching（Lipman 2023） |
|---|---|---|
| 路径形式 | $X_t=(1-t)X_0+t X_1$ | 高斯条件路径族；OT 路径 $X_t=(1-(1-\sigma_{\min})t)X_0+t X_1$ |
| $\sigma_{\min}$ | 隐含 0（端点上 Dirac，需 smoother） | 显式保留 $\sigma_{\min}>0$ |
| $\pi_0$ 假设 | **任意分布**（生成 / I2I / 域适应统一） | 默认 $\mathcal N(0,I)$ |
| coupling | **任意**，可被 rewire（[[wiki/concepts/transport-coupling]]） | 默认独立耦合，不讨论 rewire |
| 训练目标 | $\|v_\theta(X_t,t)-(X_1-X_0)\|^2$ | CFM：$\|v_\theta(X_t,t)-u_t(X_t\mid X_1)\|^2$（OT 路径上恰为 $X_1-(1-\sigma_{\min})X_0$） |
| 迭代 reflow | **构成性步骤** | 不包含 |
| 主要卖点 | reflow / 1-step / 任意 coupling | 路径设计自由度 + 与 SDE 解耦 |

> **公式同构**：取 $\sigma_{\min}=0$、独立耦合、$k=1$（不 reflow）时，RF 与 FM-OT **训练目标完全一致**。差异在**接口与流程**：RF 把 coupling 和 reflow 当一等公民；FM 把"换路径"当一等公民。两者在工业实现里（SD3/FLUX）几乎合流。

## 为什么重要（对本 wiki）

- **加速可来自训练阶段**：reflow 把 [[wiki/overview]] 推论 3 推到极限——不只改采样器（DDIM 跳步）、不只改路径（FM-OT），而是把 ODE 本身在训练阶段迭代变直。
- **工业落点**：SD3（Esser et al. 2024）与 FLUX 的训练目标即 RF 一族；其上的编辑方法（RF-Inversion, FlowEdit 等）依然继承"沿 ODE 注入条件"的范式。✅ [[wiki/methods/flux-kontext|FLUX.1 Kontext]] 是 FLUX 的统一编辑形态——但它换了条件通道（[[wiki/concepts/in-context-conditioning|in-context token 拼接]]）而非 inversion/sideband。
- **coupling 作为研究变量**：RF 把 inversion / DDIM-inv 失稳重表述为 coupling rewiring 失稳——给编辑论文一种新的诊断语言（详见 [[wiki/concepts/transport-coupling]]）。

## 关系

- 同源 / 并行：[[wiki/concepts/flow-matching]]、[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/optimal-transport-path]]、[[wiki/concepts/stochastic-interpolants|Stochastic Interpolants]]（[[wiki/sources/albergoStochasticInterpolants2023|Albergo, Boffi & Vanden-Eijnden 2023]]，✅ 已 ingest）
- **bridge SDE 对位（[[wiki/methods/ddbm|DDBM]]）**：RF 是 bridge **ODE**（确定、学速度场、直线插值），DDBM 是 bridge **SDE**（随机、学 score、[[wiki/concepts/doob-h-transform|Doob h]] 钉端点）。[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] §6.1 Case 2 证明在 noiseless 极限 $c\to0$ + 特定 VE schedule 下其 PF-ODE 漂移恰好约化为 RF 的直线项 $x_1-x_0$——但这是**有条件极限约化**，非严格包含。⚠️ DDBM Table 2 显示 RF 在**跨域低相似度** translation（DIODE）上崩盘（FID 77.18），是"OT 直线假设失效"的一条实证边界
- 核心操作：[[wiki/concepts/reflow]]、[[wiki/concepts/transport-coupling]]
- 采样近亲：[[wiki/concepts/probability-flow-ode]]（同为确定性 ODE 生成，但 score 事后导出 vs RF 直接训速度场）
- 对照：[[wiki/methods/ddim]]（diffusion 的训练 + flow 的采样）vs RF（连训练也 flow 化 + reflow 拉直）
- 作者：[[wiki/entities/xingchao-liu]]、[[wiki/entities/qiang-liu]]
- 下游模型：✅ [[wiki/methods/flux-kontext|FLUX.1 Kontext]]（已 ingest，FLUX 的统一编辑形态）；待 ingest：SD3（Esser et al. 2024）、FLUX 基础模型、InstaFlow

## 重要结果速览

- 1-rectified flow（即 $k=1$）已在 [[wiki/benchmarks/cifar10|CIFAR-10]] 与同期 FM-OT 同档；
- 2-rectified flow 把 **1-step / 2-step 采样 FID** 大幅下压，是 RF 区别于 FM 的最直观红利；
- 同框架统一覆盖无条件生成、image-to-image translation、域适应。

## 变体与扩展

- **[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|Hierarchical Rectified Flow（HRF）]]**（Zhang et al. 2025, ICLR'25）：不学期望速度，而是在 velocity space 再跑一层 RF（学加速度），用嵌套耦合 ODE 捕捉完整的多模态速度分布。轨迹可交叉、更直，低 NFE 区间有优势；但输入维度翻倍导致参数量增大，且仅在 32×32 分辨率验证。与 reflow 正交（reflow 拉直同层 ODE，HRF 加深层级）

## 待补 / 开放

- [x] ✅ overview「主要派系→flow-matching-based」首篇已由 [[wiki/methods/flux-kontext|FLUX.1 Kontext]] 填上；仍待 SD3 / FLUX 基础模型原文
- [ ] RF-Inversion 类编辑方法 ingest（是 thesis 的直接相关线）
- [ ] Reflow 收敛到 OT 的条件与速率
- [ ] RF 模型上 inversion 往返闭合的稳定性
- [ ] HRF + reflow 组合探索；HRF 在条件生成下是否缓解 mode averaging

## 出处

- [[wiki/sources/liuFlowStraightFast2022a]]（RF 原文）
- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（并行工作，formal 关系参照）
