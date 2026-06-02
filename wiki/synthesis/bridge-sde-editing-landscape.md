---
type: synthesis
title: Bridge-based 生成/编辑/加速 landscape（bridge SDE vs ODE × 任务）
aliases: [bridge landscape, bridge-sde editing landscape, 桥方法地图]
tags: [diffusion-bridge, flow-matching, image-editing, inversion, survey]
status: active
created: 2026-06-01
updated: 2026-06-02
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]", "[[wiki/sources/zhengDiffusionBridgeImplicit2025]]"]
---

# Bridge-based 生成/编辑/加速 landscape

> 由 [2026-06-01] 文献 sweep 沉淀（触发：[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] ingest）。目的：在 **bridge SDE vs bridge ODE**（[[wiki/concepts/diffusion-bridge]]）× **任务** 的网格上，标清各格分别由哪些工作占据。
> ⚠️ 下列外部论文**多数未 ingest**，仅 arXiv 链接 + 一句话定位；要用作正式依据须先精读。

## 一句话结论

- **加速 / 蒸馏格（SDE 侧）已密集**：CDBM（NeurIPS'24）/ Inverse Bridge Matching Distillation / [[wiki/sources/zhengDiffusionBridgeImplicit2025|DBIM]]（training-free，25×）。
- **inversion / 编码格（SDE 侧）**：[[wiki/sources/zhengDiffusionBridgeImplicit2025|DBIM]]（ICLR'25）以 $\rho{=}0$ 确定桥 ODE + booting noise 给出 faithful encoding / reconstruction / 插值；但**只做 reconstruction，未做 text-guided 编辑**。
- **inversion / editing 格（ODE 侧）密集**：RF-Inversion / RF-Solver-Edit / OT-for-RF-Editing。
- **统一框架**：[[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo）覆盖两列。

## 网格

| 任务 \ 动力学 | **bridge SDE（随机，学 score）** | **bridge ODE（确定，学 velocity）** |
|---|---|---|
| 生成 / translation / restoration | [[wiki/methods/ddbm\|DDBM]]、I²SB、I³SB、BBDM/EBDM、Schrödinger Bridge / Bridge-Matching | [[wiki/methods/rectified-flow\|RF]]、FM、SD3/FLUX |
| **加速 / 蒸馏** | CDBM、Inverse Bridge Matching Distillation、[[wiki/methods/dbim\|DBIM]]（训练无关，25×） | reflow / consistency / 蒸馏（极成熟） |
| **inversion / 编码重建** | [[wiki/methods/dbim\|DBIM]]（确定桥 ODE + booting noise） | RF-Inversion、RF-Solver-Edit |
| **text-guided editing** | 尚少专门工作（DBIM 只做 reconstruction） | RF-Solver-Edit、OT-for-RF-Editing |

统一框架（两列的公约数）：[[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo）。

## 关键论文定位

### 加速（SDE 侧）
- **Consistency Diffusion Bridge Models (CDBM)** · NeurIPS 2024 · [arXiv 2410.22637](https://arxiv.org/abs/2410.22637) —— 学 DDBM PF-ODE 的 consistency function，4–50× 加速；只做生成 + semantic interpolation，**不做 text-editing**。
- **Inverse Bridge Matching Distillation** · [arXiv 2502.01362](https://arxiv.org/abs/2502.01362)（2025）—— 桥匹配的蒸馏加速。

### inversion / editing（ODE 侧）
- **RF-Inversion** · ICLR 2025 · [repo](https://github.com/LituRout/RF-Inversion) —— RF 的 inversion + 风格编辑。
- **RF-Solver / RF-Edit** · ICML 2025 · [arXiv 2411.04746](https://arxiv.org/abs/2411.04746) —— 低误差 RF-ODE solver 提升 inversion 重建 + attention 特征注入编辑。
- **OT for RF Image Editing** · [arXiv 2508.02363](https://arxiv.org/abs/2508.02363) —— 统一 inversion-based 与 direct 编辑。

### DBIM（SDE 侧 inversion 原语，✅ 已 ingest 2026-06-01）
- **Diffusion Bridge Implicit Models** · ICLR 2025 · [arXiv 2405.15885](https://arxiv.org/abs/2405.15885) · THU-ML（[[wiki/entities/jun-zhu|Jun Zhu]] 组）· wiki：[[wiki/methods/dbim]] / [[wiki/sources/zhengDiffusionBridgeImplicit2025]]
- 定位：DDBM 的 DDIM 化——training-free、25× 加速；$\rho{=}0$ 确定桥 ODE + **booting noise** 给出 faithful encoding / reconstruction / 插值 = **bridge 上的 inversion 原语**。DBIM 自身只演示 reconstruction / interpolation，未做 text-guided 编辑；同组 CDBM 是其蒸馏并行。

### 相关 / 待读
- **Inverse-and-Edit** · [arXiv 2506.19103](https://arxiv.org/abs/2506.19103) —— consistency model 上的 cycle inversion editing。
- **Bidirectional Consistency Models** · [arXiv 2403.18035](https://arxiv.org/abs/2403.18035) —— 双向可逆，桥味。
- **Textualize Visual Prompt via Diffusion Bridge** · [arXiv 2501.03495](https://arxiv.org/abs/2501.03495) —— 用 bridge 做编辑，路线是 textualize visual prompt。
- [I³SB (2403.06069)](https://arxiv.org/abs/2403.06069) 已被 [[wiki/methods/dbim|DBIM]] 覆盖（DBIM 是更一般的 "DDIM for bridge"），降级为背景。

## 理论侧 landscape（2026-06-01 补 sweep）

**结论：bridge-SDE 的理论侧已基本合拢。**

- **统一与构造——完整（✅ 已 ingest 2026-06-01）**：[[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants (2303.08797)]]（自由选 interpolant → velocity+score、ODE+SDE、噪声 $\epsilon$ 可调；diffusion/RF 是特例；显式优化插值 → Schrödinger Bridge）。其 Conclusion 自指 future application：inverse problems / **inpainting** / **super-resolution** / forecasting。
- **lens 已成文**：[Diffusion Bridge or Flow Matching? A Unifying Framework and Comparative Analysis (2509.24531)](https://arxiv.org/abs/2509.24531) —— "bridge SDE vs flow ODE" 已被专门写成统一 + 对比论文。
- **Schrödinger Bridge 收敛**：DSBM(NeurIPS'23) → [IPMF 指数收敛 (2410.02601)](https://arxiv.org/abs/2410.02601) → [IMF 非渐近 KL bound (2510.20871)](https://arxiv.org/abs/2510.20871) → [离散 DDSBM ICLR'25 (2410.01500)](https://arxiv.org/abs/2410.01500)。
- **最优 stochasticity / diffusion coefficient**：[Stochasticity Control (2410.21553)](https://arxiv.org/abs/2410.21553)（"最优 coefficient 下 path-KL 与 schedule 无关"）；[Noise schedule optimal-control (2605.21911)](https://arxiv.org/abs/2605.21911)。
- **误差界**：[dimension-free error estimate (2512.01820)](https://arxiv.org/abs/2512.01820)。
- **残留开放**：tighter sample-complexity rates、discretization error 精细控制、manifold / constrained / discrete bridges。

## 关系

- 概念底座：[[wiki/concepts/diffusion-bridge]]、[[wiki/concepts/doob-h-transform]]、[[wiki/concepts/stochastic-interpolants]]
- 方法：[[wiki/methods/ddbm]]、[[wiki/methods/dbim]]、[[wiki/methods/rectified-flow]]
