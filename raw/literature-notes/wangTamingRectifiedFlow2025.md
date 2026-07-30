---
type: literature-note
citekey: wangTamingRectifiedFlow2025
title: "Taming Rectified Flow for Inversion and Editing"
aliases: ["@wangTamingRectifiedFlow2025"]
authors: "Jiangshan Wang, Junfu Pu, Zhongang Qi, Jiayi Guo, Yue Ma, Nisha Huang, Yuxin Chen, Xiu Li, Ying Shan"
firstAuthor: "Wang"
year: 2025
itemType: preprint
doi: "10.48550/arXiv.2411.04746"
url: "http://arxiv.org/abs/2411.04746"
zotero: "zotero://select/library/items/IFVPHAT2"
tags: [literature, computer-science---computer-vision-and-pattern-recognition]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-07-30
updated: 2026-07-30
ingested_to_wiki: true
wiki_page: "[[wiki/sources/wangTamingRectifiedFlow2025]]"
---

# Taming Rectified Flow for Inversion and Editing

> [!info] @wangTamingRectifiedFlow2025 · Wang et al. · 2025
> [Open in Zotero](zotero://select/library/items/IFVPHAT2) · [DOI](https://doi.org/10.48550/arXiv.2411.04746) · [URL](http://arxiv.org/abs/2411.04746) · [PDF](file:///home/lonper/Zotero/storage/LHC44QP8/Wang%20等%20-%202025%20-%20Taming%20Rectified%20Flow%20for%20Inversion%20and%20Editing.pdf)

## Abstract

> [!abstract]- Click to expand
> Rectified-flow-based diffusion transformers like FLUX and OpenSora have demonstrated outstanding performance in the field of image and video generation. Despite their robust generative capabilities, these models often struggle with inversion inaccuracies, which could further limit their effectiveness in downstream tasks such as image and video editing. To address this issue, we propose RF-Solver, a novel training-free sampler that effectively enhances inversion precision by mitigating the errors in the ODE-solving process of rectified flow. Specifically, we derive the exact formulation of the rectified flow ODE and apply the high-order Taylor expansion to estimate its nonlinear components, significantly enhancing the precision of ODE solutions at each timestep. Building upon RF-Solver, we further propose RF-Edit, a general feature-sharing-based framework for image and video editing. By incorporating self-attention features from the inversion process into the editing process, RF-Edit effectively preserves the structural information of the source image or video while achieving high-quality editing results. Our approach is compatible with any pre-trained rectified-flow-based models for image and video tasks, requiring no additional training or optimization. Extensive experiments across generation, inversion, and editing tasks in both image and video modalities demonstrate the superiority and versatility of our method. The source code is available at https://github.com/wangjiangshan0725/RF-Solver-Edit.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
-
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
%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1.
2.
3.
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：
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
> `ingest raw/literature-notes/wangTamingRectifiedFlow2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/wangTamingRectifiedFlow2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-07-30T19:02:46.099+08:00 %%
