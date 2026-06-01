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

### [2026-06-01] Bridge-SDE inversion / editing（接 FlowCycle）

- **触发源**：[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] ingest + 用户提出的组织性 lens
- **用户原创 framing**：把生成/编辑/翻译统一看成"两端分布之间的 transport"，并按实现动力学分两支——**bridge SDE**（[[wiki/methods/ddbm|DDBM]]：随机、学 score、[[wiki/concepts/doob-h-transform|Doob h]] 钉端点）vs **bridge ODE**（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]：确定、学速度场）。详见 [[wiki/concepts/diffusion-bridge]]。
- **核心 gap（选题落点）**：bridge-**ODE** 侧 inversion/editing 已成熟（RF-Inversion、用户的 FlowCycle 往返）；bridge-**SDE** 侧因有随机项，inversion 往返闭合困难、基本空白。**把 cycle-consistency / inversion 从 bridge-ODE 推广到 bridge-SDE，或统一刻画 SDE↔ODE spectrum 上 inversion 的稳定性。**
- **子问题**：
  1. 🟣 bridge-SDE inversion / editing（最近，直接复用 FlowCycle）
  2. bridge-SDE 加速（reflow/consistency 的桥版，保住 stochasticity 的多样性红利）
  3. 反向桥 stochasticity 的理论（DDBM 实验 $s\approx0.3$ 最优，无 fidelity↔diversity 理论）
  4. bridge 的 noise schedule / 参数化设计空间（扩散 schedule 文献厚，bridge 几乎空白）
- **⚠️ prior-art 边界（开题前必须 sweep，别撞车）**：
  - "bridge 可由 SDE/ODE 双视角构造"这个 lens **不是新的**——是 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo & Vanden-Eijnden 2023）已形式化的，DDBM §6 亦引。卖点必须是 **SDE 侧欠发达**，不是"我提出 lens"。
  - bridge-SDE 侧已有：DDBM、I²SB（tractable Schrödinger Bridge）、Schrödinger Bridge / Bridge-Matching（Shi 2023）、BBDM（Li 2023）——**不是 greenfield**。
  - 子问题 2（加速）**风险最高**：2024 很可能已有 consistency-bridge / 桥蒸馏工作（超出我训练数据），动手前先查。
- **【2026-06-01 sweep 结果 —— 改写判断】**
  - 🔴 **子问题 #2（bridge-SDE 加速）已被占，KILL**：Consistency Diffusion Bridge Models（CDBM，He & Zheng et al., **NeurIPS 2024**, arXiv 2410.22637）学 DDBM PF-ODE 的 consistency function，4–50× 加速；另有 Inverse Bridge Matching Distillation（arXiv 2502.01362, 2025）。不要碰。
  - 🟡 **FlowCycle 身份确认 + 关键修正**：FlowCycle = "Target-aware Image Editing via Cycle-consistent Constraints"（arXiv 2510.20212，**HKUST Long 组**）。它是 **inversion-FREE** + flow-based（bridge ODE 侧）：用 learnable noise 参数化 corruption，靠 dual cycle-consistency 学出 **target-aware 中间态**。**我之前把它当成 RF-inversion 往返是错的**——它恰恰绕开 inversion。
  - 🟡 **bridge-ODE 侧 inversion/editing 极拥挤**：RF-Inversion（ICLR'25）、RF-Solver/RF-Edit（ICML'25, 2411.04746）、Optimal Transport for RF Editing（2508.02363）。这一侧别正面进。
  - 🟢 **相对开放的格子（sharpened #1）**：把 FlowCycle 的 **target-aware cycle-consistent corruption** 从 bridge-**ODE**（确定 flow）推广到 bridge-**SDE**（DDBM/I²SB 式随机桥）。核心问题：**随机桥的中间态能否在 dual cycle-consistency 下闭合？stochasticity 是带来更好的 editability↔consistency 折中，还是破坏一致性？** 这一格暂未见正面工作（CDBM 只做加速 + semantic interpolation，不做 text-editing；diffusion-bridge editing 如 2501.03495 是 textualize visual prompt，路线不同）。
  - ⚠️ **进场前仍须细读**：Inverse-and-Edit（2506.19103，consistency model 上的 cycle inversion editing）/ Bidirectional Consistency Models（2403.18035）/ I³SB（2403.06069，"DDIM for bridge" = bridge 的确定可逆入口）/ Textualize Visual Prompt via Diffusion Bridge（2501.03495）—— 这四篇最接近，确认它们没把这格占了再动手。
- **全景地图**：[[wiki/synthesis/bridge-sde-editing-landscape]]（bridge SDE/ODE × 任务 + 理论侧现状 + 两摞优先阅读清单）
- **下一步**：精读上面 4 篇 → 确认 sharpened #1 的 gap → 升级 [[research/thesis]] 至 v0.2（把方向从泛化的"bridge-SDE inversion"收窄为"**FlowCycle 的 SDE 推广 / 随机桥上的 target-aware cycle-consistent editing**"，与 Long 组主线对齐）。

## Parked

## Killed
