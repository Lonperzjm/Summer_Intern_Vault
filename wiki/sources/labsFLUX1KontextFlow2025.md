---
type: source
title: "FLUX.1 Kontext: Flow Matching for In-Context Image Generation and Editing in Latent Space"
aliases: [FLUX.1 Kontext, FLUX Kontext, Kontext, "Black Forest Labs 2025"]
tags: [flow-matching, rectified-flow, image-editing, in-context-conditioning, latent, dit]
status: active
created: 2026-06-02
updated: 2026-06-02
raw: "[[raw/literature-notes/labsFLUX1KontextFlow2025]]"
authors: ["Black Forest Labs (Batifol, Blattmann, Boesel, Esser, Lorenz, Müller, Podell, Rombach, Sauer, …)"]
venue: "arXiv 2506.15742"
year: 2025
arxiv: "2506.15742"
---

# FLUX.1 Kontext: Flow Matching for In-Context Image Generation and Editing in Latent Space

> 文献笔记：[[raw/literature-notes/labsFLUX1KontextFlow2025]] · arXiv [2506.15742](http://arxiv.org/abs/2506.15742) · Black Forest Labs 2025

## 一句话

把图像生成与编辑**统一成一个条件分布** $p_\theta(x\mid y,c)$（$y$=上下文图像、$c$=文本指令；$y=\varnothing$ 即 T2I，$y\neq\varnothing$ 即编辑/参考生成）：在 [[wiki/methods/ldm|LDM]] 潜空间上用 [[wiki/methods/rectified-flow|rectified flow]] 训练一个 DiT，**把上下文图像编码成 latent token 与目标 token 序列拼接**（[[wiki/concepts/in-context-conditioning|in-context conditioning]]，3D RoPE 区分位置），不加噪、不走旁路分支。强项：角色一致性、多轮编辑稳定、局部/文本编辑，且比 GPT-Image-1 快约一个数量级。

## Motivation

编辑模型的两大痛点：(1) 多轮编辑里**角色/物体一致性退化**（identity drift）；(2) 每类任务（局部编辑、参考生成、风格迁移、文本编辑）各搭一套结构、不通用。FLUX.1 Kontext 主张：把"参考图 / 源图 / 视觉标记 / 文本指令"**全部当作上下文**塞进同一序列，让模型在统一注意力里自学如何用上下文——一个架构吃下所有 in-context 任务。

## Method

### 1. 建模对象：条件生成，不是 bridge

$$
p_\theta(x\mid y,c),\qquad y=\text{上下文图像（可空）},\ c=\text{文本}
$$
🟣 **关键定位（采纳用户 takeaway #1/#5）**：很多编辑问题**不必**建成显式 source→target bridge（[[wiki/methods/ddbm|DDBM]]/[[wiki/methods/dbim|DBIM]]），可直接建成**条件生成**。这是与 bridge 路线的根本分叉。

### 2. 底座：潜空间 Rectified Flow Transformer（采纳 takeaway #2）

建在 [[wiki/entities/flux|FLUX.1]] 上：图像经 autoencoder → 16-channel 潜空间 → 在潜空间做 [[wiki/concepts/flow-matching|flow matching]]。**学速度场**（不是噪声/score/bridge score）：
$$
v_\theta(z_t,t,y,c)\approx \varepsilon-x,\qquad z_t=(1-t)x+t\varepsilon
$$
架构：double-stream + single-stream DiT block 混合，factorized **3D RoPE**，Flash Attention 3。

### 3. 核心设计：上下文图像作为拼接 token（采纳 takeaway #3）

上下文图像**既不像 [[wiki/methods/sdedit|SDEdit]]/img2img 被加噪当采样起点，也不像 [[wiki/methods/controlnet|ControlNet]] 走旁路分支**；而是被编码成 latent token，与目标 token **序列拼接**，再和文本 token 一起过统一注意力。位置用 3D RoPE 区分：目标 token $u_x=(0,h,w)$、上下文 token 给虚拟时间/位置偏移 $(t,h,w)$。详见 [[wiki/concepts/in-context-conditioning]]。

### 4. 数据与加速

- 训练：收集 / 整理**数百万 relational pairs** $(x\mid y,c)$。
- 加速：**latent adversarial diffusion distillation（LADD）** → 交互级延迟。
- 变体：`[pro]`（闭源）、`[dev]`（开源权重，仅 I2I）、`[max]`（更多算力）。

## Results

KontextBench（见 [[wiki/benchmarks/kontextbench]]，1026 pairs / 5 任务）人评 + 量化：

- **速度**：T2I/I2I 延迟最低，比 GPT-Image-1 等**快约一个数量级**（Fig 7）。
- **编辑（Fig 8）**：**local editing、text editing、general CREF（角色参考）人评第一**；global editing 次于 gpt-image-1，SREF（风格参考）次于 Gen-4 References。
- **角色保持（量化）**：用 AuraFace 抽编辑前后人脸 embedding 算相似度，**Kontext 最优**（Fig 8f）。
- **多轮一致性（Fig 12）**：连续编辑下 identity drift **最慢**（AuraFace cosine 相似度衰减最缓），优于 Gen-4 / GPT-Image-high。
- **T2I**：把评测拆成 prompt following / aesthetic / realism / typography / speed 五维（抵消单一"AI 美感"偏好 = 作者称的 "bakeyness"），各维均衡、优于前代 FLUX1.1 [pro]。

## 关系（与已有 wiki 的关联）

- **本页新建**：[[wiki/methods/flux-kontext]]、[[wiki/concepts/in-context-conditioning]]、[[wiki/benchmarks/kontextbench]]、[[wiki/entities/flux]]、[[wiki/entities/black-forest-labs]]。
- **底座**：[[wiki/methods/ldm|LDM]]（潜空间）⊕ [[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]（训练目标）——正是 [[wiki/overview]] 说的"两条正交演化线在工业实现汇合"，FLUX.1 Kontext 是其编辑形态。
- **条件注入的新一档**：[[wiki/concepts/in-context-conditioning|序列拼接 in-context]] 区别于 [[wiki/concepts/cross-attention|cross-attention（文本 token K/V）]]、[[wiki/concepts/sideband-conditioning|sideband（ControlNet 旁路）]]、[[wiki/methods/sdedit|noising 起点（SDEdit）]]——给 overview 的编辑派系补一条注入通道。
- **与 bridge 路线的对照**：[[wiki/methods/ddbm|DDBM]]/[[wiki/methods/dbim|DBIM]] 把编辑/翻译建成显式随机桥；Kontext 证明工业前沿走的是**条件 FM + in-context token**，不走 bridge。🟣 这对 thesis 的方向判断很关键（见下）。
- 人物 / 机构：[[wiki/entities/robin-rombach]]、Patrick Esser、Axel Sauer 等；[[wiki/entities/black-forest-labs]]。

## 对我的 thesis 的启示

> ⚠️ 用户 thesis-implication 原文留空；以下为 Claude 识别的方向相关点，未改动 [[wiki/overview]] 工作 thesis / [[research/thesis]]。

- 🟣 **编辑前沿 ≠ bridge**：Kontext（BFL 工业旗舰）把统一编辑做成 **in-context 条件 FM**，而非 DDBM/DBIM 式 bridge。这与 [[research/ideas]] 里"bridge-SDE editing 全家桶已 KILL"的结论一致——**再次印证 bridge-editing 是支线，主线在 in-context 条件生成**。
- 🟣 **它定义了一个强 baseline + 一个 benchmark（KontextBench）**：若 thesis 要做编辑，绕不开和 Kontext 比、在 KontextBench 上测。这也意味着"纯方法新意"更难，**execution / 特定能力 / 窄 niche** 才是现实切入点（呼应止损结论）。
- 可能的窄 niche 信号：Kontext 强在角色一致性/多轮，**弱项**是 global editing、SREF（次于 gpt-image-1 / Gen-4）——这类"它没做到最好的子能力"是比"发明新范式"更现实的切入点。

## Open questions / 待追

- [ ] 🔵 序列拼接 in-context vs cross-attention 注入，在 fidelity↔editability 上的系统差异？
- [ ] LADD（latent adversarial diffusion distillation）原文（加速来源）。
- [ ] KontextBench 与既有编辑 benchmark（I2EBench / EditEval / TEdBench）的覆盖差异。
- [ ] `[dev]` 开源权重是否可作 thesis 实验底座（可复现性）。
