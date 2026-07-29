---
type: literature-note
citekey: chaTrainingFreeRefinementFlow2026
title: Training-Free Refinement of Flow Matching with Divergence-based Sampling
aliases:
  - "@chaTrainingFreeRefinementFlow2026"
authors: Yeonwoo Cha, Jaehoon Yoo, Semin Kim, Yunseo Park, Jinhyeon Kwon, Seunghoon Hong
firstAuthor: Cha
year: 2026
itemType: preprint
doi: 10.48550/ARXIV.2604.04646
url: https://arxiv.org/abs/2604.04646
zotero: zotero://select/library/items/FYXIYE4Q
tags:
  - literature
  - todo
status: read
priority: P1
my-rating: 4
created: 2026-07-28
updated: 2026-07-28
ingested_to_wiki: true
wiki_page: "[[wiki/sources/chaTrainingFreeRefinementFlow2026]]"
---

# Training-Free Refinement of Flow Matching with Divergence-based Sampling

> [!info] @chaTrainingFreeRefinementFlow2026 · Cha et al. · 2026
> [Open in Zotero](zotero://select/library/items/FYXIYE4Q) · [DOI](https://doi.org/10.48550/ARXIV.2604.04646) · [URL](https://arxiv.org/abs/2604.04646) · [PDF](file:///home/lonper/Zotero/storage/NG6FQBAG/Cha%20等%20-%202026%20-%20Training-Free%20Refinement%20of%20Flow%20Matching%20with%20Divergence-based%20Sampling.pdf)

## Abstract

> [!abstract]- Click to expand
> Flow-based models learn a target distribution by modeling a marginal velocity field, defined as the average of sample-wise velocities connecting each sample from a simple prior to the target data. When sample-wise velocities conflict at the same intermediate state, however, this averaged velocity can misguide samples toward low-density regions, degrading generation quality. To address this issue, we propose the Flow Divergence Sampler (FDS), a training-free framework that refines intermediate states before each solver step. Our key finding reveals that the severity of this misguidance is quantified by the divergence of the marginal velocity field that is readily computable during inference with a well-optimized model. FDS exploits this signal to steer states toward less ambiguous regions. As a plug-and-play framework compatible with standard solvers and off-the-shelf flow backbones, FDS consistently improves fidelity across various generation tasks including text-to-image synthesis, and inverse problems.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- A 线核心阅读：纯推理时改进 flow matching 采样质量，不改训练——与 thesis（inference-time energy-guidance for editing）方向高度一致
- divergence 作为 model-intrinsic discrepancy signal，可能与 energy-guidance 互补/叠加
%% end why-read %%

## 高亮颜色约定（个人 convention）

> 🟡 **Yellow** = 关键论点 / takeaway
> 🔴 **Red** = 我有异议 / 可疑结论 / 论文改进点
> 🟢 **Green** = 可借鉴的方法 / 公式 / trick
> 🔵 **Blue** = 后续要追溯的引用
> 🟣 **Purple** = 与我 thesis 直接相关
> ⚫ **Gray** = 背景 / 术语定义

## Annotations

%% begin annotations %%

### Imported 2026-07-28 11:16

- 🟡 **p.2** — Our core idea is to correct the trajectory at inference time rather than modifying the velocity field itself. [⤴](zotero://open-pdf/library/items/NG6FQBAG?page=2&annotation=PJ5UFH4G)

- 🟡 **p.2** — Flow Divergence Sampler (FDS) [⤴](zotero://open-pdf/library/items/NG6FQBAG?page=2&annotation=9WTI44QA)

- 🟡 **p.5** — Theorem 1. For any t such that αt ̸= 0, the optimal CFM residual satisfies  L∗  CFM(xt, t) = E  h  ∥ut(xt) − vt∥2 xt  i  = α ̇ tβt − αtβ ̇t  αt  βt∇xt · ut(xt) − β ̇td , (7)  where d is the dimensionality of the data. [⤴](zotero://open-pdf/library/items/NG6FQBAG?page=5&annotation=JS2LZRD3)

- 🟡 **p.6** — x(0) = xt, x(m) = xt + σtξ(m), ξ(m) ∼ N (0, I), m = 1, ..., M. (9) [⤴](zotero://open-pdf/library/items/NG6FQBAG?page=6&annotation=DC2WGTLU)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 问题同 HRF：FM 的 marginal velocity 在 interpolant 交叉处平均化，指向 low-density 区域，导致模糊/退化。但本文不改训练
2. **核心理论（Theorem 1）**：CFM residual $\mathcal L^\ast_\mathrm{CFM}(x_t,t)$ 可用 velocity field 的散度 $\nabla_x \cdot u_\theta(x,t)$ 显式表达——divergence 高 = discrepancy 大 = 速度平均化严重。该信号完全从 pre-trained model 计算，不需要训练数据
3. **FDS 采样**：每步在 $x_t$ 附近随机扰动 $M$ 个候选，选 divergence 最低者作为 refined state $\tilde x_t$，再积分下一步。零阶优化，无需二阶导
4. 默认 $N=1, M=1$ 就有效；仅在前半程 $t < 0.5$ 施加（交叉集中在早期）；cosine schedule $\sigma_t$
5. 实验：CIFAR-10 / ImageNet-256 全面优于 compute-matched baseline；**直接超过 HRF / VRFM 等 training-based 方法**（Table 4）；可叠加 TFG / CFG / text-to-image（SD3-M）
6. **个人统一分析**（详见 [[research/notes/2026-07-28-singularity-unified-framework]]）：奇异点处 $l \ll \Delta \ll E(d_{\min})$——模型能拟合转向但离散化可能跨过去。个人提出两种 temporal 修正策略（缩小步长精细通过 / 增大步长跨过去），与 FDS 的 spatial shift 正交，可组合
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/flow-matching]]、[[wiki/concepts/reflow]]
- 方法：[[wiki/methods/rectified-flow]]、[[wiki/methods/fmps]]
- 相关 source：[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]（HRF，同问题不同路线）
- 基线 / 对比：EDM、JiT、HRF、VRFM、SD3-Medium、TFG
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
- **高度相关，与 thesis 方向一致**（inference-time, training-free, plug-and-play）
- divergence 是 model-intrinsic 的 discrepancy signal，不需要外部 energy/reward——可与 energy-guidance 叠加（TFG+FDS 已验证）
- 启发：divergence 可作为 editing pipeline 中的 confidence map / adaptive step-size criterion
- 零阶优化在高维 latent 中可能比梯度 guidance 更实用（避免 Hessian）
- 前半程集中干预的 insight 与 editing 中 inversion 后前几步最关键的经验一致
- **建议行动**：toy experiment 对比 divergence map 与 energy gradient direction；考虑将 FDS 嵌入 editing pipeline
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ] FDS 在 editing 场景（RF-Inversion / SDEdit）中效果如何？divergence 高是否对应编辑失真？
- [ ] 零阶 refine vs 一阶 gradient guidance 的 trade-off？能否混合？
- [ ] Hutchinson estimator 在 DiT/FLUX 级别模型上的实际 wall-clock overhead？
- [ ] divergence signal 能否替代/增强 CFG 的 guidance scale 选择？
- [ ] FDS + energy-guidance 组合："where to refine"(divergence) + "how to refine"(energy)
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/chaTrainingFreeRefinementFlow2026.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/chaTrainingFreeRefinementFlow2026.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-07-28T11:17:00.446+08:00 %%
