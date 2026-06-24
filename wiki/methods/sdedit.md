---
type: method
title: SDEdit（Stochastic Differential Editing）
aliases: [SDEdit, Stochastic Differential Editing, img2img 原型, "noising-based editing"]
tags: [diffusion, editing, score-sde, noising-based, training-free, inversion-free]
status: stable
created: 2026-05-29
updated: 2026-05-29
sources: ["[[wiki/sources/mengSDEditGuidedImage2022]]"]
family: editing
---

# SDEdit（Stochastic Differential Editing）

> family 标作 `editing`：SDEdit 是 [[wiki/overview]] 主要派系 **Inversion / noising-based** 类的奠基与最朴素代表——也是第一篇真正落入 text/stroke-guided editing 范畴的方法（此前 ingest 的都是底座 / 训练目标 / 注入机制）。

## 一句话总结

**加噪到中间时刻 $t_0$ + reverse SDE**：把 guide 图 $x^{(g)}$ 加噪到 $t_0$ 当反向生成起点，用预训练 [[wiki/concepts/score-sde|score model]] 去噪回自然图像流形。零训练、零反演、零额外 loss；唯一旋钮 $t_0$（[[wiki/concepts/noising-strength|noising strength]]）滑动 realism↔faithfulness。

## 核心机制

| 组件 | 内容 |
|---|---|
| 底座 | 任意预训练 [[wiki/concepts/score-sde\|score SDE]] 模型（VE / VP / DDPM；pixel 或 latent 均可，**不绑定 SD**） |
| 输入 | guide $x^{(g)}$（三级：粗 stroke / 真图+stroke / 贴 patch）+ 超参 $t_0$ + 步数 $N$ |
| 第一步 | **单次加噪初始化**：VE 用 $x^{(g)}+\sigma(t_0)z$；VP 用 $\alpha(t_0)x^{(g)}+\sigma(t_0)\epsilon$ |
| 迭代 | 标准 reverse SDE 从 $t_0$ 跑到 $0$（[[wiki/concepts/score-sde\|Score SDE]] 公式 (4)） |
| 旋钮 | $t_0$ —— ↑ realism / ↓ faithfulness（见 [[wiki/concepts/noising-strength]]） |
| 训练 | **无**（纯采样策略） |

## Pipeline（VE-SDE，原文 Algorithm 1）

```
Require: guide x_g, t_0 (SDE 超参), N (去噪步数)
Δt ← t_0 / N
z ~ N(0, I)
x ← x_g + σ(t_0)·z              # 单次加噪初始化（guide 信息进入初始状态）
for n = N down to 1:
    t ← t_0 · n/N
    z ~ N(0, I)
    ε ← sqrt(σ²(t) − σ²(t−Δt))
    x ← x + ε²·s_θ(x, t) + ε·z   # 标准 reverse SDE 一步（Score SDE 公式 4）
return x
```

> 与 [[wiki/methods/controlnet|ControlNet]] 的关键结构差异：guide 只在**第 0 步**进入初始状态，之后是**纯 reverse**——没有"每个 $t$ 持续注入"。ControlNet 反之是全 $t$ 加性 sideband。两者是「单次介入」vs「全程介入」的两个端点。

## 适用场景与限制

**适用**：
- stroke-based image synthesis（high-level guide：粗色块）
- stroke-based image editing（mid-level guide：真图 + stroke）
- image compositing（low-level guide：贴 patch）
- 任意预训练 score / diffusion prior 上即插即用（含 SD img2img）

**限制**：
- **realism↔faithfulness 不能同时拉满**：单旋钮 $t_0$ 决定权衡位置，无法局部解耦（某区域要真、某区域要忠实做不到——这正是后来 mask-based / attention-based 编辑要解决的）
- **全局编辑**：SDEdit 对整图加噪，**不区分要改 / 要保的区域**（无 mask 时）——局部编辑能力弱
- **依赖"分布重合"前提**：guide 与目标加噪分布必须重合，否则 reverse 拉向错误流形
- **随机性**：用 SDE（每步重采 $z$）→ 同 guide 多次运行结果不同；要确定性需换 [[wiki/concepts/probability-flow-ode|PF-ODE]] 版

## Failure modes

- $t_0$ 太大 → 完全忽略 guide（退化为无条件生成）
- $t_0$ 太小 → guide 的不真实伪影没被抹掉（realism 差）
- guide 与目标域差距过大（分布不重合）→ reverse 拉不回合理图像
- 局部精确编辑（只改一个物体、保住其余）→ 无 mask 的 SDEdit 做不到

## 关联

- 出处：[[wiki/sources/mengSDEditGuidedImage2022]]
- 理论底座：[[wiki/concepts/score-sde|Score SDE]]（reverse SDE 公式、VE/VP 加噪）；[[wiki/concepts/diffusion-process|前向加噪过程]]
- 核心旋钮：[[wiki/concepts/noising-strength]]（$t_0$ / SD img2img `strength`）
- 与采样器：[[wiki/methods/ddim|DDIM]]（SD img2img 实现常用 DDIM；确定性版 SDEdit ≈ DDIM img2img）；[[wiki/concepts/probability-flow-ode|PF-ODE]]（确定性 reverse）
- 派系对照：[[wiki/methods/controlnet|ControlNet]]（全 $t$ sideband vs SDEdit 单次 $t_0$ 介入）；统一抽象 [[wiki/concepts/sideband-conditioning]]
- 编辑派系定位：[[wiki/overview]] 主要派系 → Inversion / noising-based 首篇（最朴素：不优化、不反演）
- 下游 / 后续（待 ingest）：DDIM-inversion 系、Null-text inversion、Prompt-to-Prompt（更精细的局部编辑，弥补 SDEdit 全局性短板）
- 直接扩展：[[wiki/methods/egsde|EGSDE]]（[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022|Zhao et al. 2022]]）—— SDEdit 式"加噪源图 + 目标域 reverse"是 EGSDE 的 $p_{r1}$ realism 专家，EGSDE 在其上叠加 [[wiki/concepts/energy-guidance|energy guidance]] 让源信息全程参与
- 作者：[[wiki/entities/chenlin-meng]]、[[wiki/entities/yutong-he]]、[[wiki/entities/yang-song]]、[[wiki/entities/jiaming-song]]、[[wiki/entities/jiajun-wu]]、[[wiki/entities/jun-yan-zhu]]、[[wiki/entities/stefano-ermon]]；机构 [[wiki/entities/stanford]]
