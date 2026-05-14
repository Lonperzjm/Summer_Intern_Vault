---
type: literature-note
citekey: songDenoisingDiffusionImplicit2022
title: Denoising Diffusion Implicit Models
aliases:
  - "@songDenoisingDiffusionImplicit2022"
authors: Jiaming Song, Chenlin Meng, Stefano Ermon
firstAuthor: Song
year: 2022
itemType: preprint
doi: 10.48550/arXiv.2010.02502
url: http://arxiv.org/abs/2010.02502
zotero: zotero://select/library/items/E64K6V85
tags:
  - literature
  - todo
status: unread
priority: P1
my-rating:
created: 2026-05-11
updated: 2026-05-11
ingested_to_wiki: false
wiki_page:
---

# Denoising Diffusion Implicit Models

> [!info] @songDenoisingDiffusionImplicit2022 · Song et al. · 2022
> [Open in Zotero](zotero://select/library/items/E64K6V85) · [DOI](https://doi.org/10.48550/arXiv.2010.02502) · [URL](http://arxiv.org/abs/2010.02502) · [PDF](file:///home/lonper/Zotero/storage/LVNKQK7U/Song%20等%20-%202022%20-%20Denoising%20Diffusion%20Implicit%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Denoising diffusion probabilistic models (DDPMs) have achieved high quality image generation without adversarial training, yet they require simulating a Markov chain for many steps to produce a sample. To accelerate sampling, we present denoising diffusion implicit models (DDIMs), a more efficient class of iterative implicit probabilistic models with the same training procedure as DDPMs. In DDPMs, the generative process is defined as the reverse of a Markovian diffusion process. We construct a class of non-Markovian diffusion processes that lead to the same training objective, but whose reverse process can be much faster to sample from. We empirically demonstrate that DDIMs can produce high quality samples $10 \times$ to $50 \times$ faster in terms of wall-clock time compared to DDPMs, allow us to trade off computation for sample quality, and can perform semantically meaningful image interpolation directly in the latent space.

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
### Imported 2026-05-11 19:08

- 🟡 **p.2** — In Section 3, we generalize the forward diffusion process used by DDPMs, which is Markovian, to non-Markovian ones, for which we are still able to design suitable reverse generative Markov chains. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=2&annotation=ZHPKJUUA)

- 🟡 **p.2** — In particular, we are able to use non-Markovian diffusion processes which lead to ”short” generative Markov chains (Section 4.2) that can be simulated in a small number of steps. This can massively increase sample efficiency only at a minor cost in sample quality. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=2&annotation=FYDNDHDG)

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
> `ingest raw/literature-notes/songDenoisingDiffusionImplicit2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/songDenoisingDiffusionImplicit2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-11T19:08:34.848+08:00 %%
