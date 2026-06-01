---
type: synthesis
title: Bridge-based 生成/编辑/加速 landscape（bridge SDE vs ODE × 任务）
aliases: [bridge landscape, bridge-sde editing landscape, 桥方法地图]
tags: [diffusion-bridge, flow-matching, image-editing, inversion, survey]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
---

# Bridge-based 生成/编辑/加速 landscape

> 由 [2026-06-01] 文献 sweep 沉淀（触发：[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] ingest 后用户定向 bridge-SDE）。目的：在 **bridge SDE vs bridge ODE**（[[wiki/concepts/diffusion-bridge]]）× **任务** 的网格上标清哪格被占、哪格还开，给 [[research/thesis]] / [[research/ideas]] 定位。
> ⚠️ 下列外部论文**均未 ingest**，仅 arXiv 链接 + 一句话定位；要用作正式依据须先精读。

## 一句话结论

- **加速格（SDE 侧）已满**：CDBM（NeurIPS'24）/ Inverse Bridge Matching Distillation。
- **inversion/editing 格（ODE 侧）红海**：RF-Inversion / RF-Solver-Edit / OT-for-RF-Editing / FlowCycle。
- **相对开放且与 Long 组对齐的格**：**随机桥（bridge SDE）上的 target-aware cycle-consistent editing**——即把 [[#FlowCycle（Long 组，本工作锚点）|FlowCycle]] 的范式从 ODE 推到 SDE。详见 [[research/ideas]] 2026-06-01 条。

## 网格

| 任务 \ 动力学 | **bridge SDE（随机，学 score）** | **bridge ODE（确定，学 velocity）** |
|---|---|---|
| 生成 / translation / restoration | [[wiki/methods/ddbm\|DDBM]]、I²SB、I³SB、BBDM/EBDM、Schrödinger Bridge / Bridge-Matching | [[wiki/methods/rectified-flow\|RF]]、FM、SD3/FLUX |
| **加速 / 蒸馏** | 🔴 **满**：CDBM、Inverse Bridge Matching Distillation | reflow / consistency / 蒸馏（极成熟） |
| **inversion / text-editing** | 🟢 **相对开放**（仅边缘工作，见下"待确认 4 篇"） | 🔴 **红海**：RF-Inversion、RF-Solver-Edit、OT-for-RF-Editing、[[#FlowCycle（Long 组，本工作锚点）\|FlowCycle]] |

统一框架（两列的公约数）：[[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo）。

## 关键论文定位

### 加速（SDE 侧，已占 → KILL）
- **Consistency Diffusion Bridge Models (CDBM)** · NeurIPS 2024 · [arXiv 2410.22637](https://arxiv.org/abs/2410.22637) —— 学 DDBM PF-ODE 的 consistency function，4–50× 加速；只做生成 + semantic interpolation，**不做 text-editing**。
- **Inverse Bridge Matching Distillation** · [arXiv 2502.01362](https://arxiv.org/abs/2502.01362)（2025）—— 桥匹配的蒸馏加速。

### inversion/editing（ODE 侧，红海 → 别正面进）
- **RF-Inversion** · ICLR 2025 · [repo](https://github.com/LituRout/RF-Inversion) —— RF 的 inversion + 风格编辑。
- **RF-Solver / RF-Edit** · ICML 2025 · [arXiv 2411.04746](https://arxiv.org/abs/2411.04746) —— 低误差 RF-ODE solver 提升 inversion 重建 + attention 特征注入编辑。
- **OT for RF Image Editing** · [arXiv 2508.02363](https://arxiv.org/abs/2508.02363) —— 统一 inversion-based 与 direct 编辑。

### FlowCycle（Long 组，本工作锚点）
- **Target-aware Image Editing via Cycle-consistent Constraints (FlowCycle)** · HKUST Long 组 · [arXiv 2510.20212](https://arxiv.org/abs/2510.20212) · [repo](https://github.com/HKUST-LongGroup/FlowCycle)
- 定位：**inversion-FREE** + flow-based（bridge ODE）。用 learnable noise 参数化 corruption，dual cycle-consistency 学 **target-aware 中间态**（选择性破坏 editing-relevant、保留 irrelevant）。**关键：它绕开 inversion**，不是 inversion 往返。

### 待确认 4 篇（sharpened carve 是否被占，进场前必读）
- **Inverse-and-Edit** · [arXiv 2506.19103](https://arxiv.org/abs/2506.19103) —— consistency model 上的 cycle inversion editing（最像）。
- **Bidirectional Consistency Models** · [arXiv 2403.18035](https://arxiv.org/abs/2403.18035) —— 双向可逆，桥味。
- **I³SB（Implicit I²SB）** · [arXiv 2403.06069](https://arxiv.org/abs/2403.06069) —— 非马尔可夫 I²SB = "DDIM for bridge"，bridge 的确定可逆入口。
- **Textualize Visual Prompt via Diffusion Bridge** · [arXiv 2501.03495](https://arxiv.org/abs/2501.03495) —— 用 bridge 做编辑，但路线是 textualize visual prompt。

## 对 thesis 的指向

把 [[research/ideas]] 的 sharpened #1 收窄为：**随机桥上的 target-aware cycle-consistent editing**（FlowCycle 的 SDE 推广）。核心未答问题：随机桥中间态能否在 dual cycle-consistency 下闭合？stochasticity 改善还是破坏 editability↔consistency 折中？是否升级 [[research/thesis]] v0.2 由用户定（当前选择：先不改，待 4 篇精读后再定）。

## 理论侧 landscape（2026-06-01 补 sweep）

**结论：bridge-SDE 理论已基本合拢，不宜作本科生→Long 组直博的主选题（高风险 + 离 vision/editing mission 远）。**

- **统一与构造——完整**：[Stochastic Interpolants (2303.08797)](https://arxiv.org/abs/2303.08797)（任意 interpolant → drift+score、ODE+SDE、噪声可调；+multimarginal/Riemannian/latent）。
- 🔴 **lens 已成文**：[Diffusion Bridge or Flow Matching? A Unifying Framework and Comparative Analysis (2509.24531)](https://arxiv.org/abs/2509.24531) —— "bridge SDE vs flow ODE"已被专门写成统一+对比论文，作为理论 contribution 基本被堵死。
- **Schrödinger Bridge 收敛——正在钉死**：DSBM(NeurIPS'23) → [IPMF 指数收敛 (2410.02601)](https://arxiv.org/abs/2410.02601) → [IMF 非渐近 KL bound (2510.20871)](https://arxiv.org/abs/2510.20871) → [离散 DDSBM ICLR'25 (2410.01500)](https://arxiv.org/abs/2410.01500)。
- **最优 stochasticity / diffusion coefficient——已填**：[Stochasticity Control (2410.21553)](https://arxiv.org/abs/2410.21553)（"最优 coefficient 下 path-KL 与 schedule 无关"）；[Noise schedule optimal-control (2605.21911)](https://arxiv.org/abs/2605.21911)。
- **误差界**：[dimension-free error estimate (2512.01820)](https://arxiv.org/abs/2512.01820)。
- **残留真洞（硬 + 挤，专业概率组在抢）**：tighter sample-complexity rates、discretization error 精细控制、manifold/constrained/discrete bridges。
- **若要保留理论味**：做**应用毗邻**的理论（如"随机桥上 inversion/cycle-consistency 闭合的条件"），而非通用 bridge 收敛 bound。

## 优先阅读清单（两摞，别混）

- **理论/"该不该做纯理论"**（结论已偏向"别"）：① [2303.08797 Stochastic Interpolants](https://arxiv.org/abs/2303.08797) ② [2509.24531 Bridge or Flow Matching?](https://arxiv.org/abs/2509.24531)
- **选题/编辑方向（thesis 真正要读）**：[2506.19103 Inverse-and-Edit](https://arxiv.org/abs/2506.19103) · [2403.18035 Bidirectional Consistency](https://arxiv.org/abs/2403.18035) · [2403.06069 I³SB](https://arxiv.org/abs/2403.06069) · [2501.03495 Textualize via Diffusion Bridge](https://arxiv.org/abs/2501.03495)

## 关系

- 概念底座：[[wiki/concepts/diffusion-bridge]]、[[wiki/concepts/doob-h-transform]]、[[wiki/concepts/stochastic-interpolants]]
- 方法：[[wiki/methods/ddbm]]、[[wiki/methods/rectified-flow]]
- 研究：[[research/ideas]]、[[research/thesis]]
