---
type: concept
title: In-Context Conditioning（上下文 token 序列拼接）
aliases: [in-context conditioning, 序列拼接条件, sequence concatenation conditioning, in-context editing]
tags: [conditioning, image-editing, dit, flow-matching, transformer]
status: active
created: 2026-06-02
updated: 2026-06-02
sources: ["[[wiki/sources/labsFLUX1KontextFlow2025]]"]
---

# In-Context Conditioning（上下文 token 序列拼接）

> 概念页。源：[[wiki/sources/labsFLUX1KontextFlow2025|FLUX.1 Kontext]]。

## 一句话定义

把条件（参考图 / 源图 / 视觉标记）**编码成 latent token，与目标 token 拼进同一个 Transformer 序列**，用统一注意力让模型自学如何利用上下文——而不是加噪当起点、不走旁路分支、不只改 attention map。位置由 RoPE（如 3D RoPE）区分：目标 token 给 $u_x=(0,h,w)$，上下文 token 给偏移坐标。$y=\varnothing$ 时退化为纯文本条件生成。

## 在编辑条件注入谱系中的位置

vault 既有的条件注入通道 + 本页新增：

| 通道 | 机制 | 代表 |
|---|---|---|
| noising 起点 | 源图加噪当采样初值 | [[wiki/methods/sdedit\|SDEdit]] |
| [[wiki/concepts/cross-attention\|cross-attention]] | 文本 token 作 K/V | LDM / SD |
| attention-injection | 改 cross/self-attention map | Prompt-to-Prompt 系 |
| [[wiki/concepts/sideband-conditioning\|sideband]] | 旁路 trainable copy + 加性注入 | [[wiki/methods/controlnet\|ControlNet]] |
| **in-context 拼接（本页）** | **条件图 = 序列内 latent token，统一注意力** | [[wiki/methods/flux-kontext\|FLUX.1 Kontext]] |

> 关键差异：前几类多建立在**冻结/复用**一个 T2I backbone 上外挂；in-context 拼接是把条件做进**序列本身**，需要（重）训练统一模型，但换来一个架构吃下局部/全局/参考/风格/文本/多轮编辑。

## 为什么有效（采纳 Kontext takeaway）

- **通用性**：所有 in-context 任务共享一套注意力，不必每任务设计结构。
- **一致性**：上下文图作为 token 全程在注意力里可见 → 角色/物体一致性、多轮稳定性优于"一次性重绘"路线。
- **与 bridge 的对照**：把编辑建成条件生成 $p(x\mid y,c)$，**不需要**显式 source→target [[wiki/concepts/diffusion-bridge|bridge]]。

## 与其他概念的关系

- 载体：[[wiki/concepts/flow-matching]] / [[wiki/methods/rectified-flow]]（Kontext 的训练目标）、[[wiki/methods/ldm]]（潜空间）
- 对照通道：[[wiki/concepts/cross-attention]]、[[wiki/concepts/sideband-conditioning]]
- 路线对照：[[wiki/concepts/diffusion-bridge]]（bridge）vs in-context 条件生成

## 出处与引用

- [[wiki/sources/labsFLUX1KontextFlow2025]]（FLUX.1 Kontext，序列拼接 + 3D RoPE）
