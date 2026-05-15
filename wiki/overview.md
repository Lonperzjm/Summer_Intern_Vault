---
type: synthesis
title: 领域总览 · Diffusion / Flow Matching for Text-Guided Image Editing
tags: [overview, thesis]
status: active
created: 2026-05-05
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]"]
---

# Overview

本页是整个 wiki 的"枢纽"，由 Claude Code 在 ingest 新源后持续更新。

## Working Thesis（v0.2，基于 DDPM + DDIM）

> Diffusion 模型把图像生成建模为**全局 + 渐进**的去噪过程：每一步对整张图作用（全局），但分 T 步逐级精化（渐进）。其工程化的关键在于 [[wiki/concepts/epsilon-parameterization]]——把训练目标从"预测 reverse 均值"改写为"预测注入噪声 ε"，并配合极简 L2 损失 $L_\mathrm{simple}$。这一参数化与 [[wiki/concepts/score-matching]] 在尺度因子下等价，从而把 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]] 与 [[wiki/methods/ncsn|NCSN 系（Song & Ermon 2019）]]统一在 score-based 框架下。
>
> 推论（待后续源验证 / 演化）：
>
> 1. **真正的不变量是"范式"，不是某个"层"；组件落在可变性光谱上**
>
>    与其把技术栈切成"固化层 / 编辑层"，不如承认：唯一近乎不变的是**范式**——
>    > *迭代式生成 + 网络预测噪声 / 速度场 + 沿生成链注入条件*。
>
>    其余一切组件按"被改动的频率与代价"排在一条光谱上：
>
>    | 可变性 | 组件 | 说明 |
>    |---|---|---|
>    | 近乎不变 | 上述范式本身 | 从 DDPM 到 FLUX 都成立 |
>    | 可演化，但非主战场 | 训练目标（ε → v → 速度场）、backbone（U-Net → DiT） | 改它性价比低、要和生态对抗；但**确实在变** |
>    | 研究杠杆 | inversion 质量、guidance 形式、条件注入通道、介入时间步 | 论文创新点集中于此 |
>
>    **关键修正**：原来的"基础设施层 / 编辑层"两层划分有三个硬伤——(i) 把 U-Net 当作固化组件是错的，backbone 一直在被换（DiT、SD3、FLUX）；(ii) "底层一字不改"与 Flow Matching 换训练目标自相矛盾；(iii) 最致命：两层划不开——"条件注入"这个杠杆**本身就要改 backbone 内部**（cross-attention 在 U-Net 里、ControlNet 是 U-Net 旁路副本）。所以 backbone 不是"与编辑无关的下层"，而是**研究杠杆施力的对象之一**。
>
>    **对 thesis 的含义**：不要把"重新设计训练目标"当成主战场（性价比低，要和整个生态对抗）；可行的研究杠杆是 inversion 质量、guidance 形式、条件注入通道、介入时间步——但要清醒：动"条件注入"往往意味着动 backbone 架构本身。
>
>    **支持光谱"中间一档"的证据**（下次 ingest 时继续验证）：
>    - Flow Matching / Rectified Flow 系（FLUX、SD3）**确实换了训练目标**（预测速度场而非 ε），但其上的编辑方法（RF-Inversion 等）依然继承"在 reverse/ODE 链上注入条件"的范式——这正说明"训练目标可演化、范式不变"。
>    - v-prediction、EDM preconditioning、Min-SNR weighting 等是对 $L_\mathrm{simple}$ 的"调参级"修改，能做小幅 ablation 但远未触及"重新设计目标"的级别。
>
> 2. **全局-渐进权衡**：编辑保真度 vs 可控性 trade-off 很可能与"在 reverse 链哪一段（哪一时间步）介入"强相关——高 $t$ 改变结构，低 $t$ 改变细节，这是后续要在编辑论文中重点验证的假设。
>
> 3. **采样速度是开放赛道** —— ✅ **DDIM 已部分验证**。[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 证明加速可以是**纯采样期**的事：不重训、不动 backbone，只把前向从马尔可夫松绑为非马尔可夫族（[[wiki/concepts/non-markovian-diffusion]]），就能在子序列上跳步、10–50× 加速。这把"采样加速"从训练问题降级为推断问题——对本就要反复采样的编辑场景尤其关键。进一步地，DDIM 的确定性采样（$\sigma=0$）与 ODE 反向积分给出了 **inversion** 的理论入口，直接服务于 overview「主要派系」中的 inversion-based 一类。仍待 ingest：蒸馏 / Consistency Models 对编辑质量的影响。
>
> 推论 3 同时**强化了推论 1 的可变性光谱**：DDIM 动的是"采样链的形状（步数、$\sigma$）"，属于光谱中"研究杠杆 / 介入方式"一档，且代价极低——是"范式不变、组件可调"的又一个干净样本。

> _下次 ingest 后请重新审视上述三条推论是否仍然成立。推论 1、3 已被 DDIM 正向支持；推论 2（介入时间步与 fidelity/controllability trade-off）仍待 text-guided editing 论文验证。_

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
- **采样加速线**：[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 已 ingest（验证推论 3）；下一步是扩散蒸馏 / Consistency Models / Rectified Flow few-step 采样——这些对**编辑质量**的影响仍是空白
- **训练目标的"调参级"修改**：v-prediction（Salimans & Ho 2022）、IDDPM 的 $\Sigma_\theta$ 学习、Min-SNR / EDM preconditioning —— 可作小幅 ablation，暂未值得单独成页
- **编辑方法族本体**：overview「主要派系」四类（inversion-based / instruction-tuned / attention-injection / flow-matching-based）目前全是空链接，待 ingest 第一篇 text-guided editing 论文后填充
- **DDIM inversion**：DDIM 的 ODE 可反向积分把 $x_0$ 编码回 $x_T$，已在 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]]、[[wiki/methods/ddim]]、[[wiki/concepts/non-markovian-diffusion]] 多处被提及，是 inversion-based 编辑的技术底座——暂不单独成页，待第一篇用它的编辑论文 ingest 时再建 `concepts/` 或 `methods/` 条目
- **DDIM ↔ Flow Matching 的关系（待 ingest FM 原文后正式化）**：粗记一下当前理解——DDIM 的确定性采样（$\sigma=0$）本身就是解一个 ODE（$d\bar x=\varepsilon_\theta\,d\sigma$，即 probability-flow ODE），所以 DDIM 已经有"flow 味"。区别在于：DDIM 的 ODE 是从**已训好的 ε 网络事后导出**的，不动训练；Flow Matching / Rectified Flow 则**直接把训练目标换成预测速度场**，ODE 是训出来的本体。一句话：DDIM = diffusion 的训练 + flow 的采样；FM = 连训练也 flow 化。这正好是可变性光谱"训练目标可演化、ODE 范式不变"的干净例子。待 ingest RF / FLUX / SD3 原文后再补全或单独成页。
