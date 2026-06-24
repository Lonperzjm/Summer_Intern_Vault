---
type: literature-note
citekey: yuFreeDoMTrainingFreeEnergyGuided2023b
title: "FreeDoM: Training-Free Energy-Guided Conditional Diffusion Model"
aliases: ["@yuFreeDoMTrainingFreeEnergyGuided2023b"]
authors: "Jiwen Yu, Yinhuai Wang, Chen Zhao, Bernard Ghanem, Jian Zhang"
firstAuthor: "Yu"
year: 2023
itemType: conferencePaper
doi: "10.1109/ICCV51070.2023.02118"
url: "https://ieeexplore.ieee.org/document/10377605/"
zotero: "zotero://select/library/items/DWQVP8A2"
tags: [literature]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-06-23
updated: 2026-06-23
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]"
---

# FreeDoM: Training-Free Energy-Guided Conditional Diffusion Model

> [!info] @yuFreeDoMTrainingFreeEnergyGuided2023b · Yu et al. · 2023
> [Open in Zotero](zotero://select/library/items/DWQVP8A2) · [DOI](https://doi.org/10.1109/ICCV51070.2023.02118) · [URL](https://ieeexplore.ieee.org/document/10377605/) · [PDF](file:///home/lonper/Zotero/storage/VEPD895I/Yu%20等%20-%202023%20-%20FreeDoM%20Training-Free%20Energy-Guided%20Conditional%20Diffusion%20Model.pdf)

## Abstract

> [!abstract]- Click to expand
> 

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%

- 想理解 FreeDoM 如何在不训练新的条件扩散模型、不训练 time-dependent classifier / reward model 的情况下，把现成 clean-image 判别模型接入扩散采样。
    
- 重点关注这条路线：$x_t \to \hat{x}_0(x_t,t) \to E(\hat{x}_0,c) \to \nabla_{x_t}E \to \text{guided sampling}$，判断它是否属于 clean-estimate-level energy guidance。
    
- 想比较 FreeDoM 与 DPS（待 ingest）、[[wiki/methods/egsde|EGSDE]]、[[wiki/concepts/classifier-guidance]]、[[wiki/concepts/classifier-free-guidance]]、[[wiki/methods/controlnet|ControlNet]] 的关系，尤其想确认它更接近 DPS 还是 EGSDE。
    
- 期待从这篇里得到一个更一般的研究切入点：[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/conditional-diffusion]]（clean-estimate 评分并入此页 §5）、[[wiki/concepts/energy-guidance]]、reward-guidance（待建页）。  
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

### Imported 2026-06-23 18:51

- 🟡 **p.23175** — Specifically, we leverage off-the-shelf pretrained networks, such as a face detection model, to construct time-independent energy functions, which guide the generation process without requiring training. [⤴](zotero://open-pdf/library/items/VEPD895I?page=23175&annotation=76I6UPU3)

- 🟡 **p.23175** — Once a new target condition is needed for generation, they have to retrain or finetune the models, which is inconvenient and expensive. [⤴](zotero://open-pdf/library/items/VEPD895I?page=23175&annotation=6T2K73TZ)

- 🟡 **p.23175** — we propose a sampling process guided by the energy function [⤴](zotero://open-pdf/library/items/VEPD895I?page=23175&annotation=SI85U7IS)

- 🟡 **p.23175** — we use off-theshelf pre-trained time-independent models, [⤴](zotero://open-pdf/library/items/VEPD895I?page=23175&annotation=FUFGUXP2)

- 🟡 **p.23175** — functions we construct are timeindependent and do not need to be retrained. [⤴](zotero://open-pdf/library/items/VEPD895I?page=23175&annotation=Q93263SQ)

- 🟡 **p.23176** — p(c|xt) = exp{−λE(c, xt)}  Z , [⤴](zotero://open-pdf/library/items/VEPD895I?page=23176&annotation=N8TY4XMG)

- 🟡 **p.23176** — ∇xt log p(c|xt) ∝ −∇xt E(c, xt), (4) [⤴](zotero://open-pdf/library/items/VEPD895I?page=23176&annotation=D6UG2UAK)
  - 💬 *我的批注*：不够严格，Z没算

- 🟡 **p.23177** — it is difficult to find an existing pre-trained network for noisy images. [⤴](zotero://open-pdf/library/items/VEPD895I?page=23177&annotation=NB9RLWRY)

- 🟡 **p.23177** — Dφ(c, xt, t) ≈ Ep(x0|xt)[Dθ(c, x0)]. (7) [⤴](zotero://open-pdf/library/items/VEPD895I?page=23177&annotation=J6HC23PS)

- 🟡 **p.23177** — E(c, xt) ≈ Dθ(c, x0|t). (9) [⤴](zotero://open-pdf/library/items/VEPD895I?page=23177&annotation=V7VCM48M)

- 🟡 **p.23178** — In the early stage, i.e., the chaotic stage, the generated result x0|t is extremely blurred, and the energy guidance is hard to make anything reasonable, so we do not need to employ the time-travel strategy. During the late stage, i.e., the refinement stage, the change in the generated results is minor, so the time-travel strategy is useless. During the intermediate stage, i.e., the semantic stage, the change in the generated result is significant, so this stage is critical for conditional generation. [⤴](zotero://open-pdf/library/items/VEPD895I?page=23178&annotation=GXIKVY33)

- 🟡 **p.23178** — E(c, xt) ≈ Dθ(c, x0|t) = Dist(Pθ1 (c), Pθ2 (x0|t)), (11) [⤴](zotero://open-pdf/library/items/VEPD895I?page=23178&annotation=56VUNQXA)

- 🟡 **p.23179** — E({c1, · · · , cn}, xt) ≈ η1Dθ1 (c1, x0|t) + · · · + ηnDθn (cn, x0|t), (12) [⤴](zotero://open-pdf/library/items/VEPD895I?page=23179&annotation=95GVYEJR)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%

1. FreeDoM 的核心思想是用现成的、预训练的、作用于干净图像的时间无关模型来近似原本应作用于 noisy state 的时间依赖能量函数。它不是直接对 $x_t$ 评分，而是先用扩散模型给出 clean estimate：  
    $$  
    x_{0|t}=\frac{1}{\sqrt{\bar{\alpha}_t}}\left(x_t+(1-\bar{\alpha}_t)s(x_t,t)\right),  
    $$  
    然后在 $x_{0|t}$ 上计算 $E(x_{0|t},c)$，再通过链式法则得到 $\nabla_{x_t}E$ 来修正采样。
    
2. FreeDoM 明确属于 clean-estimate-level energy guidance，结构上更接近 DPS（待 ingest），而不是 [[wiki/methods/egsde|EGSDE]]。DPS 是 $x_t \to \hat{x}_0 \to E(\hat{x}_0;y) \to \nabla_{x_t}E$，FreeDoM 则把观测一致性能量推广为更一般的条件能量，例如 CLIP 文本距离、分割距离、草图距离、ArcFace 身份距离、Gram 风格距离和低通结构距离。
    
3. FreeDoM 的 training-free 含义是任务层面的免训练：不训练 diffusion model，不训练新的 time-dependent classifier，不训练 ControlNet 式控制分支，而是在采样时使用已有模型构造能量。但它并不是“什么模型都不依赖”，而是依赖已有的无条件扩散模型和现成 clean-image scoring / perception models。
    
4. FreeDoM 的采样公式可以概括为：  
    $$  
    x_{t-1}=m_t-\rho_t\nabla_{x_t}E(x_{0|t}(x_t),c).  
    $$  
    对潜变量扩散则是：  
    $$  
    z_t \to z_{0|t} \to \hat{x}_0=\operatorname{Dec}(z_{0|t}) \to E(\hat{x}_0,c) \to \nabla_{z_t}E.  
    $$  
    因此它可以外挂到 Stable Diffusion、ControlNet 等已有训练式条件模型上，形成“训练式内部条件接口 + 免训练外部能量接口”的混合控制方式。
    
5. 这篇的局限也很关键：FreeDoM 省训练成本但增加推理成本，因为每步需要额外能量反传，time-travel 还会增加采样步；在 ImageNet 这类大数据域中，细粒度结构控制较弱，Canny 边缘这类条件即使用 time-travel 也可能 poor guidance；多条件时简单加权假设条件独立，遇到条件冲突会产生较差结果。这些局限正好指向后续研究方向：更高效、更稳定、能处理多条件冲突的 training-free / low-training clean-estimate-level energy guidance。  
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/energy-guidance]]、[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/classifier-guidance]]
- 方法：[[wiki/methods/freedom]]、[[wiki/methods/egsde]]、[[wiki/methods/controlnet]]、[[wiki/methods/ldm]]
- 实体（作者 / 模型 / 机构）：Jiwen Yu、Jian Zhang（PKU）、Bernard Ghanem（KAUST）
- 基线 / 对比：DPS（最近邻，待 ingest）、TFG（统一框架，待 ingest）
- 本篇 wiki source 页：[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/yuFreeDoMTrainingFreeEnergyGuided2023b.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-23T18:51:51.035+08:00 %%
