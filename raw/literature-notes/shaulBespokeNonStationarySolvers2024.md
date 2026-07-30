---
type: literature-note
citekey: shaulBespokeNonStationarySolvers2024
title: "Bespoke Non-Stationary Solvers for Fast Sampling of Diffusion and Flow Models"
aliases: ["@shaulBespokeNonStationarySolvers2024"]
authors: "Neta Shaul, Uriel Singer, Ricky T. Q. Chen, Matthew Le, Ali Thabet, Albert Pumarola, Yaron Lipman"
firstAuthor: "Shaul"
year: 2024
itemType: preprint
doi: "10.48550/arXiv.2403.01329"
url: "http://arxiv.org/abs/2403.01329"
zotero: "zotero://select/library/items/83NSHYBR"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---artificial-intelligence, computer-science---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-07-30
updated: 2026-07-30
ingested_to_wiki: true
wiki_page: "[[wiki/sources/shaulBespokeNonStationarySolvers2024]]"
---

# Bespoke Non-Stationary Solvers for Fast Sampling of Diffusion and Flow Models

> [!info] @shaulBespokeNonStationarySolvers2024 · Shaul et al. · 2024
> [Open in Zotero](zotero://select/library/items/83NSHYBR) · [DOI](https://doi.org/10.48550/arXiv.2403.01329) · [URL](http://arxiv.org/abs/2403.01329) · [PDF](file:///home/lonper/Zotero/storage/5IMFHZK5/Shaul%20等%20-%202024%20-%20Bespoke%20Non-Stationary%20Solvers%20for%20Fast%20Sampling%20of%20Diffusion%20and%20Flow%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> This paper introduces Bespoke Non-Stationary (BNS) Solvers, a solver distillation approach to improve sample efficiency of Diffusion and Flow models. BNS solvers are based on a family of non-stationary solvers that provably subsumes existing numerical ODE solvers and consequently demonstrate considerable improvement in sample approximation (PSNR) over these baselines. Compared to model distillation, BNS solvers benefit from a tiny parameter space ($<$200 parameters), fast optimization (two orders of magnitude faster), maintain diversity of samples, and in contrast to previous solver distillation approaches nearly close the gap from standard distillation methods such as Progressive Distillation in the low-medium NFE regime. For example, BNS solver achieves 45 PSNR / 1.76 FID using 16 NFE in class-conditional ImageNet-64. We experimented with BNS solvers for conditional image generation, text-to-image generation, and text-2-audio generation showing significant improvement in sample approximation (PSNR) in all.

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
> `ingest raw/literature-notes/shaulBespokeNonStationarySolvers2024.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/shaulBespokeNonStationarySolvers2024.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-07-30T19:01:25.611+08:00 %%
