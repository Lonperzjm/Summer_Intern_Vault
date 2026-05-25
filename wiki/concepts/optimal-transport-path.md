---
type: concept
title: Optimal Transport (OT) Conditional Path
aliases: [OT path, OT 路径, optimal transport path, OT 条件路径, 最优传输路径]
tags: [flow-matching, optimal-transport, ode]
status: active
created: 2026-05-24
updated: 2026-05-24
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]"]
---

# Optimal Transport (OT) Conditional Path

> 概念页。源：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023]]。是 [[wiki/concepts/flow-matching|Flow Matching]] 在高斯条件路径里的一个**非 diffusion** 选择。

## 一句话定义

在 [[wiki/concepts/conditional-flow-matching|CFM]] 的高斯条件路径 $p_t(x\mid x_1)=\mathcal N(\mu_t,\sigma_t^2 I)$ 里，取**均值线性、方差线性**的设计，使噪声→数据走**直线、匀速**——这恰是两高斯之间的最优传输 displacement map。

## 公式

$$\mu_t(x_1)=t\,x_1,\qquad \sigma_t(x_1)=1-(1-\sigma_{\min})t.$$
$$x_t=\big(1-(1-\sigma_{\min})t\big)x_0+t\,x_1,\qquad
u_t(x\mid x_1)=\frac{x_1-(1-\sigma_{\min})x}{1-(1-\sigma_{\min})t}.$$
代入 $x=x_t$，条件速度化为**恒定**向量
$$\frac{d}{dt}\psi_t(x_0)=x_1-(1-\sigma_{\min})x_0.$$
方向与大小都不随 $t$ 变 → 轨迹是直线。

## 对比 diffusion 路径

| | diffusion 路径（VP/VE） | OT 路径 |
|---|---|---|
| 轨迹 | 弯曲 | **直线** |
| 去噪节奏 | 只在末段才去噪 | 近似**线性**全程去噪 |
| 速度场 | 随 $t$ 变 | 恒定 |
| NFE / 训练速度 | 较多 / 较慢 | **更少 / 更快** |
| 泛化 | — | 更好（论文报告） |

> 直觉：diffusion 让样本"先在噪声里打转、最后才显形"，OT 让它"一路匀速从噪声滑向数据"。

## 实测收益（来自原文）

同架构下 FM-OT 在 CIFAR-10 / ImageNet-32/64 的 NLL、FID、NFE **三项一致最好**（见 [[wiki/sources/lipmanFlowMatchingGenerative2023#Results]]）；达到同等采样误差只需约 60% 的 NFE。

## 与其他概念的关系

- 框架：[[wiki/concepts/flow-matching]] / [[wiki/concepts/conditional-flow-matching]]
- 对立面：diffusion 路径（[[wiki/methods/ddpm]] VP / [[wiki/methods/ncsn]] VE）
- 下游：[[wiki/methods/rectified-flow|Rectified Flow]]（Liu et al. 2022）把"直线化"推到极致，是 SD3 / FLUX 训练目标的来源（待 ingest）

## 在 text-guided editing 中的作用

- 直线路径 + 低 NFE 对反复采样的编辑场景友好；其对 inversion 往返闭合的影响是开放问题（对照 [[wiki/methods/ddim]] failure mode）。

## 出处与引用

- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（OT 条件路径，eq 22–24）
- 概念根：McCann 1997（高斯间 OT displacement interpolation）
