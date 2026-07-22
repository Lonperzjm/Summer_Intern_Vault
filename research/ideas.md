---
type: research
title: Idea 池
status: active
created: 2026-05-05
updated: 2026-06-24
---

# Ideas

<!-- 候选 idea 暂存。每条注明触发源（[[sources/...]]）与日期。 -->

## Active

### [2026-06-18 · 更新 2026-06-29] Energy-guided conditional generation —— 公开文献无 carve，存活仅靠导师在研线

来源：师兄 6/2 推 [EGSDE (NeurIPS'22)](https://arxiv.org/abs/2207.06635)（范式 **discriminative logits/reward → energy → guidance**）；6/23 定 [[wiki/methods/freedom|FreeDoM]]=baseline + 三段改进框架。已 ingest：[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022|EGSDE]]、[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b|FreeDoM]]、[[wiki/sources/songFlowMatchingPosterior2025|FMPS]]。全景：[[wiki/synthesis/energy-guidance-landscape]]。

**核心 novelty 假设**：怎么从 $x_t$ 高效且准确拿 $x_0$ 评分，再用 energy 转引导。

**✅ sweep 三轮（关键词 + 引用图 + 第一性原理），结论：generic 全红，比 bridge-SDE 还挤。**

**🎯 师兄 6/23 三段框架 —— 逐段 sweep 后全部沦陷：**

| 段 | 原理化方向 | 结局 |
|---|---|---|
| ① $\hat x_0$ 估计 | RF $\hat x_0=x_t-tv$ | ☠️ **死**：[[wiki/sources/songFlowMatchingPosterior2025\|FMPS]] 占（gradient/free 两实现都给了）；且 $\hat x_0=\mathbb E[x_0\mid x_t]$ 也是后验均值，"flow 更准"理论存疑（[Straightness is not your need](https://arxiv.org/html/2410.07303v2)）；况且能高效得 $x_0$ 是比 energy 更大的鱼，scope 错配 |
| ② 结构化 E | 保/丢分解、现成模型拼、正交不打架 | 🔴 **占**：[TtfDiffusion](https://www.sciencedirect.com/science/article/abs/pii/S0925231224019301) / [DICE](https://arxiv.org/html/2602.08059)（结构-语义解耦）+ [GradOPS](https://arxiv.org/html/2503.03438v1)（正交消冲突） |
| ③ →guidance | 原理化步长/流形/便宜雅可比 | 🔴 **占**：MPGD/TFG/manifold-CFG + FMPS 的 $g^1$ 归一化 + free 版便宜雅可比 |

**❌ 收束作废**：原"换 flow 底座盘活三段"已塌——① flow 不更准、③ flow 上 FMPS 已占。**flow 不是金底座。**

**引用图硬证据**：FreeDoM 的引用里"沾 RF/flow"的一抓一把（FlowChef 32 引领跑），高引后代全往 test-time reward 对齐汇 → 领域重心已过、饱和。

**存活仅一条**：坐师兄/Long 组 flow 在研线（红海三铁律唯一命中项）。

**下一步**：把"①②③全红 + 已查 FMPS/FlowChef/TtfDiffusion/DICE/GradOPS"地图带回师兄，让他用**非公开信息**定具体 execution sliver；**不再 armchair carve**（已证多次必撞）。第一个实验设计待 sliver 定。
**待 ingest（🔵）**：DPS、[TFG 2403.12404](https://arxiv.org/abs/2403.12404)（统一框架）、MPGD（③ 的流形改法）。

---

> **[2026-06-01] 当前无存活的"方法级" idea。** bridge/flow/diffusion editing 是 2024–2026 最红的红海，连撞三次（见 Killed）。教训：**不在最热的方法层 armchair 找题**。下一步不是再想一个 carve，而是：
> 1. **更窄的 niche**（具体域/模态/评测空白），离红海远一点；
> 2. **导师 brief**——方向得和 Long 组导师定，把这几周的地图 + 三次撞车结论交回该接的人。
>
> 全景地图：[[wiki/synthesis/bridge-sde-editing-landscape]]。

## Parked

（暂无）

## Killed

### [2026-06-01] "让 SDE bridge 脱离 diffusion 框架"（理论方向）—— KILL（撞 Stochastic Interpolants）

- 想法：把随机桥做成独立对象，直接在两分布间定义、不从 VP/VE 扩散 + Doob h 推。
- 🔴 **撞车**：= [[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants (Albergo 2023)]] 本身（自由选插值/任意两分布/ODE+SDE 可调噪声/还原 SB 全中）。该框架更早更一般，DDBM 反而是它"绑回扩散"的特例。
- 结论：理论 contribution 已满，放弃。直觉正确（"DDBM 依赖扩散是 crutch"），但被占。

### [2026-06-01] "Bridge-SDE editing 全家桶"（含 diversity 旋钮 / 加速 / inversion）—— KILL（三次 sweep 三次撞）

源头 framing（仍有教学价值，但每个落点都被占）：把生成/编辑/翻译统一为"两端分布 transport"，bridge SDE（[[wiki/methods/ddbm|DDBM]]）vs bridge ODE（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]），见 [[wiki/concepts/diffusion-bridge]]。逐个落点的撞车记录：

- **bridge-SDE 加速** → 🔴 [CDBM (NeurIPS'24, 2410.22637)](https://arxiv.org/abs/2410.22637)、[Inverse Bridge Matching Distillation (2502.01362)](https://arxiv.org/abs/2502.01362)。
- **bridge inversion / encoding** → 🔴 [[wiki/methods/dbim|DBIM (ICLR'25)]] 已给确定可逆桥 + booting noise 原语。
- **bridge-ODE inversion/editing** → 🔴 红海：RF-Inversion (ICLR'25)、[RF-Solver-Edit (2411.04746)](https://arxiv.org/abs/2411.04746)、[OT-for-RF-Editing (2508.02363)](https://arxiv.org/abs/2508.02363)。
- **可控多样性 / 随机 flow 编辑（diversity 旋钮）** → 🔴 [OSCAR: Orthogonal Stochastic Control for diversity in Flow Matching (2510.09060)](https://arxiv.org/abs/2510.09060)（training-free 注入正交随机扰动提多样性、不损 fidelity）；Variational Rectified Flow Matching（latent 捕获多模）；Discretized-RF（stochastic velocity 补多样性）；[FlowSlider (2604.02088)](https://arxiv.org/abs/2604.02088)（fidelity-steering 旋钮）。one-to-many 编辑自 Blended Diffusion (2022) 即有。
- **stochasticity / schedule 理论** → 🔴 SB/SI + [Stochasticity Control (2410.21553)](https://arxiv.org/abs/2410.21553)（"最优 coefficient 下 path-KL 与 schedule 无关"）。

**meta 教训**：armchair 想方法 → sweep 查 → 撞，循环三次即证明方法错。红海里出论文靠 execution + 窄 niche + 导师在研线，不靠 blue-sky 新意。
