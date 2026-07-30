---
type: literature-note
citekey: shaulBespokeSolversGenerative2023
title: "Bespoke Solvers for Generative Flow Models"
aliases: ["@shaulBespokeSolversGenerative2023"]
authors: "Neta Shaul, Juan Perez, Ricky T. Q. Chen, Ali Thabet, Albert Pumarola, Yaron Lipman"
firstAuthor: "Shaul"
year: 2023
itemType: preprint
doi: "10.48550/arXiv.2310.19075"
url: "http://arxiv.org/abs/2310.19075"
zotero: "zotero://select/library/items/TUBKMZF9"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---artificial-intelligence, computer-science---machine-learning]
status: read            # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-07-30
updated: 2026-07-30
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/shaulBespokeSolversGenerative2023]]"
---

# Bespoke Solvers for Generative Flow Models

> [!info] @shaulBespokeSolversGenerative2023 · Shaul et al. · 2023
> [Open in Zotero](zotero://select/library/items/TUBKMZF9) · [DOI](https://doi.org/10.48550/arXiv.2310.19075) · [URL](http://arxiv.org/abs/2310.19075) · [PDF](file:///home/lonper/Zotero/storage/QXSVB4WR/Shaul%20等%20-%202023%20-%20Bespoke%20Solvers%20for%20Generative%20Flow%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Diffusion or flow-based models are powerful generative paradigms that are notoriously hard to sample as samples are defined as solutions to high-dimensional Ordinary or Stochastic Differential Equations (ODEs/SDEs) which require a large Number of Function Evaluations (NFE) to approximate well. Existing methods to alleviate the costly sampling process include model distillation and designing dedicated ODE solvers. However, distillation is costly to train and sometimes can deteriorate quality, while dedicated solvers still require relatively large NFE to produce high quality samples. In this paper we introduce "Bespoke solvers", a novel framework for constructing custom ODE solvers tailored to the ODE of a given pre-trained flow model. Our approach optimizes an order consistent and parameter-efficient solver (e.g., with 80 learnable parameters), is trained for roughly 1% of the GPU time required for training the pre-trained model, and significantly improves approximation and generation quality compared to dedicated solvers. For example, a Bespoke solver for a CIFAR10 model produces samples with Fréchet Inception Distance (FID) of 2.73 with 10 NFE, and gets to 1% of the Ground Truth (GT) FID (2.59) for this model with only 20 NFE. On the more challenging ImageNet-64$\times$64, Bespoke samples at 2.2 FID with 10 NFE, and gets within 2% of GT FID (1.71) with 20 NFE.

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

### Imported 2026-07-30 16:39

- 🟡 **p.4** — φr(x) = srx, and its inverse φ−1  r (x) = x/sr, (14) [⤴](zotero://open-pdf/library/items/QXSVB4WR?page=4&annotation=LSLERECJ)

- 🟡 **p.4** — u ̄r(x) = s ̇r  sr  x + t ̇rsrutr  x sr  . [⤴](zotero://open-pdf/library/items/QXSVB4WR?page=4&annotation=GVGDLTID)

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
> `ingest raw/literature-notes/shaulBespokeSolversGenerative2023.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/shaulBespokeSolversGenerative2023.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-07-30T16:40:09.925+08:00 %%
