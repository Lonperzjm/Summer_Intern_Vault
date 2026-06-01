---
type: method
title: DDBM（Denoising Diffusion Bridge Models）
aliases: [DDBM, Denoising Diffusion Bridge Models, 扩散桥模型, diffusion bridge model]
tags: [diffusion-bridge, image-translation, score-based, doob-h-transform]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"]
family: bridge
---

# DDBM（Denoising Diffusion Bridge Models）

> 方法主页。原文 [[wiki/sources/zhouDenoisingDiffusionBridge2023|Zhou, Lou, Khanna & Ermon 2023]] 已 ingest。本页负责 DDBM 在 vault 中的"方法族入口"：它把 vault 的脉络从 noise→data 的扩散，扩到 **paired-distribution translation 的随机桥**一支。

## 一句话

从任意 VP/VE 扩散过程出发，用 [[wiki/concepts/doob-h-transform|Doob's h-transform]] 把过程**钉死在终点** $x_T=y$ 得到 forward bridge SDE；只学桥的 score $s=\nabla_{x_t}\log q(x_t\mid x_T)$，沿反向 SDE / [[wiki/concepts/probability-flow-ode|PF-ODE]] 从 $y$ 生成 $x_0$。

## 核心配方（四步）

1. **选扩散过程**：VP 或 VE，给出 $f,g$、转移核 $p(x_t\mid x_0)$、$\mathrm{SNR}_t=\alpha_t^2/\sigma_t^2$；
2. **h-transform 钉终点**：forward bridge SDE $\mathrm dx_t=[f+g^2 h]\mathrm dt+g\,\mathrm dw_t$，$h=\nabla_{x_t}\log p(x_T{=}y\mid x_t)$；
3. **闭式桥分布**：$q(x_t\mid x_0,x_T)\propto p(x_T\mid x_t)p(x_t\mid x_0)$ 为高斯（公式 8），提供训练标签；
4. **学 score + 采样**：denoising bridge score matching（公式 9）回归条件 score；hybrid sampler（Heun ODE + 调度 Euler-Maruyama SDE 步 + guidance $w$）反向生成。

## 与相邻方法的关系

| | DDBM（bridge SDE） | [[wiki/methods/rectified-flow\|Rectified Flow]]（bridge ODE） | [[wiki/methods/sdedit\|SDEdit]]（noising） |
|---|---|---|---|
| 动力学 | 随机 SDE（+ PF-ODE 可选） | 确定 ODE | 反向 SDE/ODE |
| 端点 | Doob 钉死 $x_T=y$ | 直线插值两端 | 加噪起点 $t_0$ |
| 学什么 | 桥 score $s$ | 速度场 $v$ | 复用无条件 score |
| 任务 | paired translation + 生成 | 生成 + I2I | editing（单旋钮 $t_0$） |

- **退化关系**：源端设高斯 → 严格回 [[wiki/concepts/score-sde|score diffusion]]（§6.1 Case 1）；noiseless 极限 $c\to0$ + 特定 VE schedule → 约化到 OT-FM/RF（§6.1 Case 2，**有条件，非严格特例**）。
- **统一归宿**：bridge SDE 与 bridge ODE 的严格统一在 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]，DDBM 用的是不同的 denoising bridge score-matching loss。
- **复用扩散设计**：EDM preconditioning、higher-order（Heun）sampler、VP/VE schedule 全部直接搬用——这是 DDBM 相对 Schrödinger-Bridge 类方法的工程红利。

## 重要结果速览

- **I2I translation（pixel）**：Edges→Handbags-64 FID **1.83**（VP）/ DIODE-256 FID **4.43**（VP），断层超过 Pix2Pix / SDEdit / RF / DDIB，优于最接近的 I²SB（7.43 / 9.34）。
- **跨域鲁棒**：RF 在低层相似度低的 DIODE 上崩到 77.18，DDBM 随机桥不受影响。
- **无条件生成**：源端=高斯时 CIFAR-10 FID 2.06（≈EDM 2.04）、FFHQ-64 FID 2.44（<EDM 2.53）。

## 关系网

- 源：[[wiki/sources/zhouDenoisingDiffusionBridge2023]]
- 机制：[[wiki/concepts/doob-h-transform]]、[[wiki/concepts/infinitesimal-generator]]、[[wiki/concepts/diffusion-bridge]]
- 快速采样 / 隐式桥：[[wiki/methods/dbim|DBIM]]（DDBM 的 DDIM 化，training-free、最高 25× 加速、$\rho{=}0$ 确定可逆桥支持 encoding/reconstruction）；CDBM（consistency 蒸馏加速）
- 近亲：[[wiki/concepts/stochastic-interpolants]]、[[wiki/methods/rectified-flow]]、[[wiki/concepts/flow-matching]]、[[wiki/methods/sdedit]]
- 采样/训练复用：[[wiki/concepts/probability-flow-ode]]、[[wiki/concepts/predictor-corrector-sampling]]、[[wiki/concepts/score-matching]]
- 作者：[[wiki/entities/linqi-zhou]]、[[wiki/entities/aaron-lou]]、[[wiki/entities/stefano-ermon]]（[[wiki/entities/stanford]]）
- landscape：[[wiki/synthesis/bridge-sde-editing-landscape]]（bridge SDE/ODE × 任务地图 + 理论侧现状）

## 待补 / 开放

- [ ] I²SB / Schrödinger Bridge 类对照页
- [ ] EDM（Karras 2022）独立页（DDBM 的 preconditioning 与 sampler 基础）
- [ ] DDBM 上做 text-guided editing / inversion 的后续工作（与 FlowCycle 接合）——inversion 原语已由 [[wiki/methods/dbim|DBIM]] 提供（确定 ODE + booting noise），开口在其上做 text-guided 编辑
