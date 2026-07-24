---
type: literature-note
citekey: peeblesScalableDiffusionModels2023
title: "Scalable Diffusion Models with Transformers"
aliases: ["@peeblesScalableDiffusionModels2023"]
authors: "William Peebles, Saining Xie"
firstAuthor: "Peebles"
year: 2023
itemType: preprint
doi: "10.48550/arXiv.2212.09748"
url: "http://arxiv.org/abs/2212.09748"
zotero: "zotero://select/library/items/CCXL2NI6"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-07-24
updated: 2026-07-24
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/peeblesScalableDiffusionModels2023]]"
---

# Scalable Diffusion Models with Transformers

> [!info] @peeblesScalableDiffusionModels2023 · Peebles et al. · 2023
> [Open in Zotero](zotero://select/library/items/CCXL2NI6) · [DOI](https://doi.org/10.48550/arXiv.2212.09748) · [URL](http://arxiv.org/abs/2212.09748) · [PDF](file:///home/lonper/Zotero/storage/U2SRSGGJ/Peebles和Xie%20-%202023%20-%20Scalable%20Diffusion%20Models%20with%20Transformers.pdf)

## Abstract

> [!abstract]- Click to expand
> We explore a new class of diffusion models based on the transformer architecture. We train latent diffusion models of images, replacing the commonly-used U-Net backbone with a transformer that operates on latent patches. We analyze the scalability of our Diffusion Transformers (DiTs) through the lens of forward pass complexity as measured by Gflops. We find that DiTs with higher Gflops -- through increased transformer depth/width or increased number of input tokens -- consistently have lower FID. In addition to possessing good scalability properties, our largest DiT-XL/2 models outperform all prior diffusion models on the class-conditional ImageNet 512x512 and 256x256 benchmarks, achieving a state-of-the-art FID of 2.27 on the latter.

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

### Imported 2026-07-24 11:36

- 🟡 **p.3** — Diffusion [⤴](zotero://open-pdf/library/items/U2SRSGGJ?page=3&annotation=63GUQMKD)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 潜变量切成 patch
2. 编码时间步和类别条件
3. Attention很多轮
4. 输出 tokens 还原为空间预测
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
> `ingest raw/literature-notes/peeblesScalableDiffusionModels2023.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/peeblesScalableDiffusionModels2023.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-07-24T11:36:12.035+08:00 %%
