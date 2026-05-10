---
type: literature-note
citekey: hoDenoisingDiffusionProbabilistic2020
title: "Denoising Diffusion Probabilistic Models"
aliases: ["@hoDenoisingDiffusionProbabilistic2020"]
authors: "Jonathan Ho, Ajay Jain, Pieter Abbeel"
firstAuthor: "Ho"
year: 2020
itemType: preprint
doi: "10.48550/arXiv.2006.11239"
url: "http://arxiv.org/abs/2006.11239"
zotero: "zotero://select/library/items/QWX957DH"
tags: [literature, todo]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-05-10
updated: 2026-05-10
ingested_to_wiki: false # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page:              # e.g. "[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"
---

# Denoising Diffusion Probabilistic Models

> [!info] @hoDenoisingDiffusionProbabilistic2020 · Ho et al. · 2020
> [Open in Zotero](zotero://select/library/items/QWX957DH) · [DOI](https://doi.org/10.48550/arXiv.2006.11239) · [URL](http://arxiv.org/abs/2006.11239) · [PDF](file:///home/lonper/Zotero/storage/ZDDVEK6X/Ho%20等%20-%202020%20-%20Denoising%20Diffusion%20Probabilistic%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> We present high quality image synthesis results using diffusion probabilistic models, a class of latent variable models inspired by considerations from nonequilibrium thermodynamics. Our best results are obtained by training on a weighted variational bound designed according to a novel connection between diffusion probabilistic models and denoising score matching with Langevin dynamics, and our models naturally admit a progressive lossy decompression scheme that can be interpreted as a generalization of autoregressive decoding. On the unconditional CIFAR10 dataset, we obtain an Inception score of 9.46 and a state-of-the-art FID score of 3.17. On 256x256 LSUN, we obtain sample quality similar to ProgressiveGAN. Our implementation is available at https://github.com/hojonathanho/diffusion

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

-

## 高亮颜色约定（个人 convention）

> 🟡 **Yellow** = 关键论点 / takeaway
> 🔴 **Red** = 我有异议 / 可疑结论 / 论文改进点
> 🟢 **Green** = 可借鉴的方法 / 公式 / trick
> 🔵 **Blue** = 后续要追溯的引用
> 🟣 **Purple** = 与我 thesis 直接相关
> ⚫ **Gray** = 背景 / 术语定义

## Annotations

绝对的开山之作，将diffusion引入主流视野的paper.
### Imported 2026-05-05 20:58

- 🟡 **p.1** — Denoising Diffusion Probabilistic Models [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=1&annotation=7N9NQUFB)

### Imported 2026-05-09 16:24

- 🟡 **p.1** — Denoising Diffusion Probabilistic Models [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=1&annotation=ZM2AIS6G)

- 🟢 **p.1** — a progressive lossy decompression scheme [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=1&annotation=6KNV2PKY)

- 🟡 **p.2** — they are capable of generating high quality samples. [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=XE69JLBH)

- 🟡 **p.2** — In addition, we show that a certain parameterization of diffusion models reveals an equivalence with denoising score matching over multiple noise levels during training and with annealed Langevin dynamics during sampling (Section 3.2) [55, 61]. [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=ZPTJLA6K)

- 🔴 **p.2** — We find that the majority of our models’ lossless codelengths are consumed to describe imperceptible image details (Section 4.3). [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=C7X5EDK9)

### Imported 2026-05-09 16:37

- 🟡 **p.9** — Since diffusion models seem to have excellent inductive biases for image data, [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=9&annotation=D8NHWH2A)

- 🔴 **p.9** — we look forward to investigating their utility in other data modalities and as components in other types of generative models and machine learning systems. [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=9&annotation=SDGYPWY7)
  - 💬 *我的批注*：晶体生成，for example

### Imported 2026-05-10 14:49

- 🟡 **p.2** — We find that the majority of our models’ lossless codelengths are consumed to describe imperceptible image details (Section 4.3). [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=MUJ6MTZR)

- 🟢 **p.2** — E [− log pθ(x0)] ≤ Eq  [  − log pθ(x0:T )  q(x1:T |x0)  ]  = Eq  [  − log p(xT ) − ∑  t≥1  log pθ(xt−1|xt)  q(xt|xt−1)  ]  =: L (3) [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=XUN58BGB)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

1. 文章做了什么
	1. 证明 diffusion model 可以生成高质量图像
	2. 提出/强调一种有效的参数化方式：预测噪声
	3. 建立 diffusion model 和 score-based 方法的联系
2. looking forward
	1. 其他数据模态中的用途（晶体生成等）
	2. 作为组件在其他模型中 
3. 基本机制  
	1. Forward process：  
		逐步给真实图像加高斯噪声，直到信号被破坏。  
	2. Reverse process：  
		学习反向去噪过程，从纯噪声逐步生成图像。  
	3. 训练时：  
		输入带噪图像 xt 和时间步 t，预测噪声 epsilon。  
	4. 采样时：  
		从高斯噪声 xT 开始，逐步去噪得到 x0。

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

-

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

- [ ]

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/hoDenoisingDiffusionProbabilistic2020.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-10T14:49:28.120+08:00 %%
