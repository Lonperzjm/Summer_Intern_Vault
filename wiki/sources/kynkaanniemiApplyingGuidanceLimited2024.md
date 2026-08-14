---
type: source
title: "Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"
aliases: [Guidance Interval, Limited Interval Guidance, "Kynkäänniemi et al. 2024"]
tags: [diffusion, guidance, classifier-free-guidance, sampling, inference-efficiency]
status: active
created: 2026-08-11
updated: 2026-08-14
raw: "[[raw/literature-notes/kynkaanniemiApplyingGuidanceLimited2024]]"
authors: "Tuomas Kynkäänniemi, Miika Aittala, Tero Karras, Samuli Laine, Timo Aila, Jaakko Lehtinen"
venue: NeurIPS 2024
year: 2024
arxiv: "2404.07724"
---

# Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models

> Kynkäänniemi et al., NeurIPS 2024。核心结论不是“把 guidance 开得更强”，而是**只在合适的噪声区间开启 guidance**：高噪声阶段有害，中间阶段有益，低噪声阶段基本无必要。

## Motivation

传统 [[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance（CFG）]] 在整条 reverse sampling chain 上使用固定 guidance scale。这隐含假设：条件引导在所有 noise level 上同样有益。

作者直接检验这一假设，发现 guidance 的效果具有明显的三阶段结构：

| 采样阶段 | 噪声水平 | Guidance 效果 |
|---|---:|---|
| 初期 | 高 | 有害，分布质量下降 |
| 中期 | 中 | 明显有益 |
| 后期 | 低 | 基本无必要 |

因此，CFG 的问题不只在 guidance scale，也在 **timing**。

## Method

以 conditional prediction 为基线，将通常的 CFG 写成：

$$
D_w(x;\sigma,c)=D_c(x;\sigma,c)+w\left[D_c(x;\sigma,c)-D_u(x;\sigma)\right].
$$

作者把常数 $w$ 改成 noise-dependent 的矩形窗：

$$
w(\sigma)=
\begin{cases}
w, & \sigma_{\mathrm{lo}}\le \sigma\le \sigma_{\mathrm{hi}},\\
0, & \text{otherwise}.
\end{cases}
$$

也就是只在中间 noise interval 计算 conditional 与 unconditional 两个分支；区间外退回单次 conditional prediction。该修改：

- 不改网络与训练目标；
- 不需要重新训练；
- 只新增 interval 上下界两个采样超参数；
- 区间外省去 unconditional forward，因此可以降低推理成本。

它可视为最简单的 **time/noise-dependent guidance scheduler**。

## Results

- **ImageNet 512×512**：当时的最佳 FID 从 **1.81 降至 1.40**，只改变推理时 guidance 的作用区间。
- 改善并非局限于单一配置：作者报告其在不同 sampler 参数、网络架构和数据集上均有定量或定性收益。
- 在大规模 text-to-image 模型 **Stable Diffusion XL** 上也观察到定性改善，说明它不只是 ImageNet class-conditional 模型的局部 trick。
- 关闭早期与后期 guidance 还减少双分支前向次数，使质量改善与推理节省可以同时出现。

### 结果边界

- FID 是分布级指标，不能单独证明 condition fidelity、diversity 与感知质量全部同步改善。
- 最优 interval 仍是模型、噪声参数化和 sampler 相关的超参数，论文没有从理论上直接推出通用边界。
- “高噪声阶段过早锁定语义、低噪声阶段条件与无条件方向趋同”是解释实验现象的有用直觉，不应写成已严格证明的机制定理。

## 关系

- **核心概念**：[[wiki/concepts/classifier-free-guidance]]。本文把 CFG 从单一 scale 旋钮扩展为“**scale × noise interval**”两个维度。
- **前身**：[[wiki/sources/hoClassifierFreeDiffusionGuidance2022|Ho & Salimans 2022]] 定义全程 CFG；[[wiki/sources/dhariwalDiffusionModelsBeat2021|Dhariwal & Nichol 2021]] 给出 classifier guidance 的质量—多样性权衡。
- **连续时间语言**：在 [[wiki/concepts/score-sde]] 中，guidance 是逐时间加入的条件 correction；本文证明该 correction 的经验价值并非常数。
- **后续自动化**：[[wiki/methods/learn-to-guide|Learn to Guide]] 从 self-consistency 学习 condition- 与 $(s,t)$-dependent 权重；Limited Interval 是其手工 schedule baseline。
- **模型与底座**：在 [[wiki/methods/ldm|latent diffusion]] / [[wiki/entities/stable-diffusion|Stable Diffusion]] 系统中，interval 与 prompt CFG、sampler、noise schedule 可组合，但时间坐标与阈值需要按具体参数化解释。
- **与编辑旋钮的对照**：[[wiki/concepts/noising-strength|SDEdit 的 noising strength]] 决定 reverse chain 的起点；guidance interval 决定已进入 reverse chain 后在哪一段施加条件 correction。两者可组合，但是否近似正交仍需实验。

## 对我的 thesis 的启示

1. **不改变 working thesis 版本号。** 当前主线仍是 [[wiki/synthesis/reject-and-skip-research-direction|Reject-and-Skip]]；本文提供的是对 overview“介入时间步是低成本研究杠杆”的直接支持，而非方向级替换。

2. **时间位置与引导强度应分开建模。** CFG 不能再只记为一个标量 scale；更合理的实验变量是 correction 的 magnitude、support interval 与 sampler/noise coordinate。

3. **对 flow / trajectory 研究的接口。** 若把 guidance 写成
   $$v_{\mathrm{guided}}(x,t)=v_c(x,t)+w(t)\big(v_c(x,t)-v_u(x,t)\big),$$
   本文的实证说明 correction 的收益强烈依赖 $t$。值得研究的不是简单照搬矩形窗，而是能否用 curvature、conditional–unconditional disagreement 或状态可信度自适应决定“何时引导”。这与 Reject-and-Skip 的事件触发思想有方法论相似性，但目前只是研究假设，不是已有证据。

## 待调研方向

- [ ] 能否从 score、posterior 或 trajectory geometry 推导 interval，而不是网格搜索 $\sigma_{\mathrm{lo}},\sigma_{\mathrm{hi}}$？
- [ ] [[wiki/methods/learn-to-guide|Learn to Guide]] 已在 1.05B [[wiki/concepts/flow-matching|Flow Matching]] 模型上验证 learned weight；矩形 window 如何跨 time parameterization 对齐仍待比较。
- [ ] 在 text-guided editing 中，guidance interval 与 noising strength、attention injection、ControlNet sideband 是否近似正交？
- [ ] FID 改善究竟来自 early-stage distribution coverage、late-stage compute removal，还是二者共同作用？需用 precision/recall、prompt adherence 与逐阶段消融拆解。

## 出处

- 原始文献笔记：[[raw/literature-notes/kynkaanniemiApplyingGuidanceLimited2024]]
- arXiv: 2404.07724
- NeurIPS 2024 Main Conference
