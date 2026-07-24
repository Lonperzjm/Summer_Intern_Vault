---
type: method
title: FLUX.1 Kontext
aliases: [FLUX.1 Kontext, FLUX Kontext, Kontext]
tags: [flow-matching, rectified-flow, image-editing, in-context-conditioning, dit, latent]
status: active
created: 2026-06-02
updated: 2026-06-02
sources: ["[[wiki/sources/labsFLUX1KontextFlow2025]]"]
family: editing
---

# FLUX.1 Kontext

> 方法主页。原文 [[wiki/sources/labsFLUX1KontextFlow2025|Black Forest Labs 2025]] 已 ingest。统一图像生成 + 编辑的工业旗舰模型。

## 一句话

[[wiki/entities/flux|FLUX.1]] 潜空间 [[wiki/methods/rectified-flow|rectified flow]] [[wiki/sources/peeblesScalableDiffusionModels2023\|DiT]]，把生成/编辑统一为条件分布 $p_\theta(x\mid y,c)$，靠 **[[wiki/concepts/in-context-conditioning|上下文图像 token 序列拼接]]** 注入条件——不加噪、不走旁路。

## 核心配方

1. **底座**：autoencoder → 16-ch 潜空间；double+single stream DiT；3D RoPE；Flash Attention 3。
2. **训练目标**：rectified flow，$v_\theta(z_t,t,y,c)\approx\varepsilon-x$，$z_t=(1-t)x+t\varepsilon$。
3. **条件注入**：上下文图像 → latent token，与目标 token **拼接**进同一序列，RoPE 位置偏移区分（目标 $u_x=(0,h,w)$）；文本 token 同序列。$y=\varnothing$ → T2I。
4. **数据**：数百万 $(x\mid y,c)$ relational pairs。
5. **加速**：LADD（latent adversarial diffusion distillation）→ 交互级延迟。变体 `[pro]`/`[dev]`（开源，I2I）/`[max]`。

## 与相邻编辑方法的分界

| 方法 | 条件怎么进模型 | backbone |
|---|---|---|
| [[wiki/methods/sdedit\|SDEdit]] | 源图加噪当采样起点 | 复用无条件 score |
| [[wiki/methods/controlnet\|ControlNet]] | 旁路 trainable copy + zero-conv | 冻结主干 + sideband |
| attention-injection（P2P 等） | 改 cross/self-attention map | 冻结主干 |
| **FLUX.1 Kontext** | **上下文图像 = 拼接 latent token，统一注意力** | **微调统一 DiT** |

> 关键：前三类多在**冻结/复用**一个 T2I 模型上外挂；Kontext 是**重新训练一个统一 DiT**，把 in-context 条件做进序列本身。详见 [[wiki/concepts/in-context-conditioning]]。

## 重要结果（KontextBench）

- 速度最低延迟（比 GPT-Image-1 快约一个数量级）；
- local / text editing / 角色参考 (CREF) 人评第一，AuraFace 角色保持第一；
- 多轮编辑 identity drift 最慢；
- global editing / 风格参考 (SREF) 次于 gpt-image-1 / Gen-4。

详见 [[wiki/benchmarks/kontextbench]]。

## 关系网

- 源：[[wiki/sources/labsFLUX1KontextFlow2025]]
- 底座：[[wiki/methods/ldm]]、[[wiki/methods/rectified-flow]]、[[wiki/concepts/flow-matching]]、[[wiki/entities/flux]]
- 条件机制：[[wiki/concepts/in-context-conditioning]]（vs [[wiki/concepts/cross-attention]] / [[wiki/concepts/sideband-conditioning]]）
- 对照（非 bridge）：[[wiki/methods/ddbm]]、[[wiki/methods/dbim]]
- benchmark：[[wiki/benchmarks/kontextbench]]
- 机构 / 人物：[[wiki/entities/black-forest-labs]]、[[wiki/entities/robin-rombach]]

## 待补 / 开放

- [ ] LADD 原文（加速底层）
- [ ] `[dev]` 开源权重作为 thesis 实验底座的可复现性
- [ ] in-context 拼接 vs cross-attention 的 fidelity↔editability 系统对比
