---
type: synthesis
title: 领域总览 · Diffusion / Flow Matching for Text-Guided Image Editing
tags: [overview, thesis]
status: active
created: 2026-05-05
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Overview

本页是整个 wiki 的"枢纽"，由 Claude Code 在 ingest 新源后持续更新。

## Working Thesis（v0.1，基于 DDPM 一篇）

> Diffusion 模型把图像生成建模为**全局 + 渐进**的去噪过程：每一步对整张图作用（全局），但分 T 步逐级精化（渐进）。其工程化的关键在于 [[wiki/concepts/epsilon-parameterization]]——把训练目标从"预测 reverse 均值"改写为"预测注入噪声 ε"，并配合极简 L2 损失 $L_\mathrm{simple}$。这一参数化与 [[wiki/concepts/score-matching]] 在尺度因子下等价，从而把 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]] 与 [[wiki/methods/ncsn|NCSN 系（Song & Ermon 2019）]]统一在 score-based 框架下。
>
> 推论（待后续源验证 / 演化）：
>
> 1. **基础设施层固化，差异空间在"编辑层"**：把 diffusion editing 的技术栈画成两层——
>    - **基础设施层**：ε-prediction + $L_\mathrm{simple}$ + U-Net + sinusoidal time embedding；
>    - **编辑层**：(a) inversion（如何把真实图像映回噪声链）、(b) guidance（[[wiki/concepts/classifier-free-guidance|CFG]] 及其变体）、(c) 条件注入（cross-attention、instruction tuning、ControlNet 旁路、attention map 替换…）、(d) 介入时间步（在 reverse 链的哪个 $t$ 开始/结束施加条件）。
>
>    从 DDPM (2020) 到 Stable Diffusion 再到 InstructPix2Pix / Prompt2Prompt / Null-text Inversion / ControlNet / SDEdit，**底层一字不改**——大家都用同一个 ε-pred 训练目标、要么直接复用 SD 权重，要么用同一个目标自训。所有论文的创新点都落在编辑层。
>
>    **对 thesis 的含义**：不要把"重新设计训练目标"当成主战场（性价比低，要和整个生态对抗）；可行的研究杠杆是**编辑层的四个旋钮**——介入时间步、注入通道、inversion 质量、guidance 形式。
>
>    **已知 caveat**（下次 ingest 时验证 / 改写）：
>    - Flow Matching / Rectified Flow 系（FLUX、SD3）**确实换了训练目标**（预测速度场而非 ε），但其上的编辑方法（RF-Inversion 等）依然继承"在 reverse/ODE 链上注入条件"的范式。所以更精确的说法或许是：**训练目标本身可演化，但"在生成链上注入条件"才是编辑的核心动作**。
>    - v-prediction、EDM preconditioning、Min-SNR weighting 等是对 $L_\mathrm{simple}$ 的"调参级"修改，能做小幅 ablation 但远未触及"重新设计目标"的级别。
>
> 2. **全局-渐进权衡**：编辑保真度 vs 可控性 trade-off 很可能与"在 reverse 链哪一段（哪一时间步）介入"强相关——高 $t$ 改变结构，低 $t$ 改变细节，这是后续要在编辑论文中重点验证的假设。
>
> 3. **采样速度是开放赛道**：DDPM 默认 T=1000，编辑场景下耗时尤甚；DDIM / 蒸馏 / Consistency 等加速方案对编辑质量的影响是值得专门 ingest 的方向。

> _下次 ingest 后请重新审视上述三条推论是否仍然成立。_

## 当前关注的子问题

- 编辑保真度（fidelity）vs 可控性（controllability）trade-off
- 全局编辑 vs 局部编辑的统一框架
- Flow Matching / Rectified Flow 相对 score-based diffusion 在编辑任务上的优劣
- Inversion 质量对编辑结果的影响
- 评测：CLIP-based 指标的局限与替代

## 主要派系（待填充链接）

- Inversion-based：…
- Instruction-tuned：…
- Attention/feature-injection：…
- Flow-matching-based：…

## 关键源

```dataview
LIST title
FROM "wiki/sources"
SORT updated DESC
LIMIT 20
```

## 待调研方向

> 由 [2026-05-14] lint 填充。来源：现有 wiki 页中反复出现但尚无独立条目 / 标注"待补"的术语。

- **score-based 主线的两个上游**：[[wiki/methods/diffusion-2015|Sohl-Dickstein 2015]]（diffusion 原型）与 [[wiki/methods/ncsn|NCSN (Song & Ermon 2019)]] —— 已建 stub，待 ingest 原文厘清与 DDPM 的精确差异
- **连续时间统一框架**：[[wiki/concepts/score-sde|Score SDE (Song et al. 2021)]] —— 严格化"small-$\beta$ ⇒ 反向高斯"的离散近似，并打通 probability-flow ODE → DDIM / Flow Matching 的脉络
- **guidance 旋钮**：[[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance]] 及其前身 classifier guidance（Dhariwal & Nichol 2021）—— 编辑层四旋钮之一，待 ingest 原文
- **采样加速线**：DDIM 已有待读文献笔记 [[raw/literature-notes/songDenoisingDiffusionImplicit2022]]（`status: unread`），是验证推论 3"采样速度是开放赛道"的下一篇
- **训练目标的"调参级"修改**：v-prediction（Salimans & Ho 2022）、IDDPM 的 $\Sigma_\theta$ 学习、Min-SNR / EDM preconditioning —— 可作小幅 ablation，暂未值得单独成页
- **编辑方法族本体**：overview「主要派系」四类（inversion-based / instruction-tuned / attention-injection / flow-matching-based）目前全是空链接，待 ingest 第一篇 text-guided editing 论文后填充
