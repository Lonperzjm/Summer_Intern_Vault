---
type: research
title: Idea 池
status: active
created: 2026-05-05
updated: 2026-06-01
---

# Ideas

<!-- 候选 idea 暂存。每条注明触发源（[[sources/...]]）与日期。 -->

## Active

> **[2026-06-01] 当前无存活的"方法级" idea。** bridge/flow/diffusion editing 是 2024–2026 最红的红海，连撞三次（见 Killed）。教训：**不在最热的方法层 armchair 找题**。下一步不是再想一个 carve，而是：
> 1. **FlowCycle 弱点分析**——"修一个具体的洞"比"想一个大新意"好找、且对口实验室代码；
> 2. **更窄的 niche**（具体域/模态/评测空白），离红海远一点；
> 3. **导师 brief**——方向得和 Long 组导师定，把这几周的地图 + 三次撞车结论交回该接的人。
>
> 全景地图：[[wiki/synthesis/bridge-sde-editing-landscape]]。

## Parked

（暂无）

## Killed

### [2026-06-01] "让 SDE bridge 脱离 diffusion 框架"（理论方向）—— KILL（撞 Stochastic Interpolants）

- 想法：把随机桥做成独立对象，直接在两分布间定义、不从 VP/VE 扩散 + Doob h 推。
- 🔴 **撞车**：= [[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants (Albergo 2023)]] 本身（自由选插值/任意两分布/ODE+SDE 可调噪声/还原 SB 全中）。该框架更早更一般，DDBM 反而是它"绑回扩散"的特例。
- 结论：理论 contribution 已满，放弃。直觉正确（"DDBM 依赖扩散是 crutch"），但被占。

### [2026-06-01] "Bridge-SDE editing 全家桶"（含 FlowCycle-SDE / diversity 旋钮 / 加速 / inversion）—— KILL（三次 sweep 三次撞）

源头 framing（仍有教学价值，但每个落点都被占）：把生成/编辑/翻译统一为"两端分布 transport"，bridge SDE（[[wiki/methods/ddbm|DDBM]]）vs bridge ODE（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]），见 [[wiki/concepts/diffusion-bridge]]。逐个落点的撞车记录：

- **bridge-SDE 加速** → 🔴 [CDBM (NeurIPS'24, 2410.22637)](https://arxiv.org/abs/2410.22637)、[Inverse Bridge Matching Distillation (2502.01362)](https://arxiv.org/abs/2502.01362)。
- **bridge inversion / encoding** → 🔴 [[wiki/methods/dbim|DBIM (ICLR'25)]] 已给确定可逆桥 + booting noise 原语。
- **bridge-ODE inversion/editing** → 🔴 红海：RF-Inversion (ICLR'25)、[RF-Solver-Edit (2411.04746)](https://arxiv.org/abs/2411.04746)、[OT-for-RF-Editing (2508.02363)](https://arxiv.org/abs/2508.02363)。
- **可控多样性 / 随机 flow 编辑（diversity 旋钮）** → 🔴 [OSCAR: Orthogonal Stochastic Control for diversity in Flow Matching (2510.09060)](https://arxiv.org/abs/2510.09060)（training-free 注入正交随机扰动提多样性、不损 fidelity）；Variational Rectified Flow Matching（latent 捕获多模）；Discretized-RF（stochastic velocity 补多样性）；[FlowSlider (2604.02088)](https://arxiv.org/abs/2604.02088)（fidelity-steering 旋钮）。one-to-many 编辑自 Blended Diffusion (2022) 即有。
- **stochasticity / schedule 理论** → 🔴 SB/SI + [Stochasticity Control (2410.21553)](https://arxiv.org/abs/2410.21553)（"最优 coefficient 下 path-KL 与 schedule 无关"）。

**meta 教训**：armchair 想方法 → sweep 查 → 撞，循环三次即证明方法错。红海里出论文靠 execution + 窄 niche + 导师在研线，不靠 blue-sky 新意。
