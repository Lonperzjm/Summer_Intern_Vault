---
type: method
title: DBIM（Diffusion Bridge Implicit Models）
aliases: [DBIM, Diffusion Bridge Implicit Models, 隐式扩散桥]
tags: [diffusion-bridge, sampling-acceleration, non-markovian, inversion]
status: active
created: 2026-06-01
updated: 2026-06-01
sources: ["[[wiki/sources/zhengDiffusionBridgeImplicit2025]]"]
family: bridge
---

# DBIM（Diffusion Bridge Implicit Models）

> 方法主页。原文 [[wiki/sources/zhengDiffusionBridgeImplicit2025|Zheng et al. 2025 (ICLR)]] 已 ingest。DBIM 是 [[wiki/methods/ddbm|DDBM]] 的**训练无关快速采样器 + 确定可逆桥**，与 DDBM 的关系完全平行于 [[wiki/methods/ddim|DDIM]] 之于 [[wiki/methods/ddpm|DDPM]]。

## 一句话

把 DDBM 的桥松绑成一族**保边缘的[[wiki/concepts/non-markovian-diffusion|非马尔可夫]]桥** → 复用 DDBM 的 bridge score 不重训 → $\rho$ 控随机性、子序列跳步，最多 **25×** 加速；$\rho{=}0$ 给出确定性桥 ODE + booting noise，实现 faithful encoding / reconstruction / 插值。

## 核心算法

1. 用 DDBM 训好的 $x_\theta(x_t,t,x_T)$（或 score $s_\theta$），无需新训练；
2. 预测 $\hat x_0$ → 反推 predicted bridge noise $\hat\varepsilon=(x_{t_{n+1}}-a_{t_{n+1}}x_T-b_{t_{n+1}}\hat x_0)/c_{t_{n+1}}$；
3. 更新 $x_{t_n}=a_{t_n}x_T+b_{t_n}\hat x_0+\sqrt{c_{t_n}^2-\rho_n^2}\,\hat\varepsilon+\rho_n\varepsilon$（公式 15）；
4. $\rho_n=0$ → 确定 ODE（可上高阶 solver，≤3 阶）；初始步注入 **booting noise**（$c_T{=}0$ 奇异）作 latent。

## 与相邻方法

| | DBIM | [[wiki/methods/ddbm\|DDBM]] | [[wiki/synthesis/bridge-sde-editing-landscape\|CDBM]] |
|---|---|---|---|
| 加速方式 | training-free 隐式（非马尔可夫族） | —（基座，hybrid sampler） | consistency 蒸馏/训练 |
| 重训 | 否 | — | 是 |
| 确定/随机 | $\rho$ 可调，$\rho{=}0$ 确定 | hybrid（含随机步） | 可少步 |
| inversion/编码 | ✅（$\rho{=}0$ + booting noise） | ✗（hybrid 有随机步） | 部分（语义插值） |
| NFE | ~20 即超 baseline | 100–500 | 4–50× 加速 |

- 结构对位：**DBIM:DDBM ＝ DDIM:DDPM**（见 [[wiki/concepts/non-markovian-diffusion]]）。
- 同组并行：CDBM（NeurIPS'24，蒸馏）；DBIM（ICLR'25，隐式）——共享作者 [[wiki/entities/kaiwen-zheng|Kaiwen Zheng]]。

## 重要结果速览

- **translation**：Edges→Handbags DBIM@20 FID **1.74**（超 DDBM@118 1.83）；DIODE DBIM@100 FID **2.57**（超 DDBM@200 3.34）。
- **inpainting（ImageNet-256）**：DBIM@20 > DDBM@500（**25×**）；$\eta{=}0$ @100 步 FID **3.91**。
- **stochasticity**：translation 确定性 $\eta{=}0$ 最优；inpainting 大 NFE 时 $\eta{=}0.8\sim1$ 最优（多样性）。$\eta{=}0$ 独占 encoding/reconstruction 能力。

## 关系网

- 源：[[wiki/sources/zhengDiffusionBridgeImplicit2025]]
- 基座：[[wiki/methods/ddbm]]；结构对位 [[wiki/concepts/non-markovian-diffusion]]、[[wiki/methods/ddim]]
- 采样：[[wiki/concepts/probability-flow-ode]]（桥 ODE）
- 概念：[[wiki/concepts/diffusion-bridge]]、[[wiki/concepts/score-matching]]
- landscape / thesis：[[wiki/synthesis/bridge-sde-editing-landscape]]、[[research/ideas]]
- 作者：[[wiki/entities/kaiwen-zheng]]、[[wiki/entities/jun-zhu]]（[[wiki/entities/tsinghua-university]]）

## 待补 / 开放

- [ ] text-guided editing on DBIM（原文只做 reconstruction/interpolation，未做 text 编辑——thesis 开口）
- [ ] 高阶桥 solver 推导
- [ ] DBIM vs CDBM 在 inversion/编辑上的权衡
