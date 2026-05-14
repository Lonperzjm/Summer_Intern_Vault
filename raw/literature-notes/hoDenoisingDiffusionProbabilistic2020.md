---
type: literature-note
citekey: hoDenoisingDiffusionProbabilistic2020
title: Denoising Diffusion Probabilistic Models
aliases:
  - "@hoDenoisingDiffusionProbabilistic2020"
authors: Jonathan Ho, Ajay Jain, Pieter Abbeel
firstAuthor: Ho
year: 2020
itemType: preprint
doi: 10.48550/arXiv.2006.11239
url: http://arxiv.org/abs/2006.11239
zotero: zotero://select/library/items/QWX957DH
tags:
  - literature
  - todo
status: read
priority: P1
my-rating: "5"
created: 2026-05-10
updated: 2026-05-11
ingested_to_wiki: true
wiki_page: "[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"
---

# Denoising Diffusion Probabilistic Models

> [!info] @hoDenoisingDiffusionProbabilistic2020 · Ho et al. · 2020
> [Open in Zotero](zotero://select/library/items/QWX957DH) · [DOI](https://doi.org/10.48550/arXiv.2006.11239) · [URL](http://arxiv.org/abs/2006.11239) · [PDF](file:///home/lonper/Zotero/storage/ZDDVEK6X/Ho%20等%20-%202020%20-%20Denoising%20Diffusion%20Probabilistic%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> We present high quality image synthesis results using diffusion probabilistic models, a class of latent variable models inspired by considerations from nonequilibrium thermodynamics. Our best results are obtained by training on a weighted variational bound designed according to a novel connection between diffusion probabilistic models and denoising score matching with Langevin dynamics, and our models naturally admit a progressive lossy decompression scheme that can be interpreted as a generalization of autoregressive decoding. On the unconditional CIFAR10 dataset, we obtain an Inception score of 9.46 and a state-of-the-art FID score of 3.17. On 256x256 LSUN, we obtain sample quality similar to ProgressiveGAN. Our implementation is available at https://github.com/hojonathanho/diffusion

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 了解diffusion是为什么流行的，初步来看，是因为他即是全局的，也是渐进的。
- 了解基础
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
绝对的开山之作，将diffusion引入主流视野的paper.主要改进了reverse的方式（改成去噪），有点像resnet。

### Imported 2026-05-10 21:55

- 🔵 **p.2** — 2 Background [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=4SQQRPJ7)
  - 💬 *我的批注*：古老的高斯输入x_t与t,输出μθ​(xt​,t),Σθ​(xt​,t)，讲一下怎么算loss,还挺麻烦的

- 🟢 **p.2** — E [− log pθ(x0)] ≤ Eq  [  − log pθ(x0:T )  q(x1:T |x0)  ]  = Eq  [  − log p(xT ) − ∑  t≥1  log pθ(xt−1|xt)  q(xt|xt−1)  ]  =: L (3) [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=XUN58BGB)
  - 💬 *我的批注*：log是凸的

- 🟢 **p.2** — reparameterization [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=L2EPMX44)

- 🔴 **p.2** — and expressiveness of the reverse process is ensured in part by the choice of Gaussian conditionals in pθ(xt−1|xt), because both processes have the same functional form when βt are small [53]. [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=2&annotation=GJ6UE53P)
  - 💬 *我的批注*：SDE理论问题

- 🟢 **p.3** — q(xt−1|xt, x0) = N (xt−1;  ̃μt(xt, x0), β ̃tI), (6)  where  ̃μt(xt, x0) :=  √α ̄t−1βt 1 − α ̄t  x0 +  √αt(1 − α ̄t−1)  1 − α ̄t  xt and β ̃t := 1 − α ̄t−1  1 − α ̄t  βt (7) [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=3&annotation=2STCPYAJ)
  - 💬 *我的批注*：inverse is Gaussians

- ⚫ **p.3** — Ex0,  [  1 2σt2  ∥ ∥ ∥ ∥  √1αt  (  xt(x0, ) − βt  √1 − α ̄t  )  − μθ(xt(x0, ), t)  ∥ ∥ ∥ ∥  2  ] [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=3&annotation=NACSK4DQ)
  - 💬 *我的批注*：公式7参与推导

- 🔴 **p.5** — However, we found it beneficial to sample quality (and simpler to implement) to train on the following variant of the variational bound [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=5&annotation=W8RDAPEB)

- 🟡 **p.5** — Lsimple(θ) := Et,x0,  [∥ ∥ − θ(√α ̄tx0 + √1 − α ̄t , t)∥  ∥  2  ] [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=5&annotation=YL8J6AGS)
### Imported 2026-05-11 16:31

- 🟡 **p.6** — 4.3 Progressive coding [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=6&annotation=3L5SF487)

### Imported 2026-05-11 17:08

- 🟡 **p.6** — 4.3 Progressive coding [⤴](zotero://open-pdf/library/items/ZDDVEK6X?page=6&annotation=3L5SF487)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
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
		1. 从数据集中取真实图像 x0
		2. 随机采样时间步 t
		3. 采样高斯噪声 ε
		4. 构造 xt = sqrt(alpha_bar_t)x0 + sqrt(1-alpha_bar_t)ε
		5. 把 xt 和 t 输入 U-Net
		6. 模型输出 εθ(xt,t)
		7. 用 MSE 让 εθ 接近真实噪声 ε  
	4. 采样时：  
		1. 从随机噪声 xT 开始
		2. 模型预测当前噪声图里的噪声
		3. 根据预测噪声去掉一部分噪声
		4. 再加入适当随机性
		5. 重复 1000 步左右
		6. 得到清晰图像 x0。
4. 具体算法 ![[hoDenoisingDiffusionProbabilistic2020-1778421475909.webp]]
5. 学会了用渐进与全局的思想生成图像，感受到resnet叔叔强大的适配性。
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/diffusion-process]]、[[wiki/concepts/epsilon-parameterization]]、
  [[wiki/concepts/variational-bound-elbo]]、[[wiki/concepts/score-matching]]、
  [[wiki/concepts/langevin-dynamics]]、[[wiki/concepts/reparameterization-trick]]
- 方法：[[wiki/methods/ddpm]]
- 实体：[[wiki/entities/jonathan-ho]]、[[wiki/entities/pieter-abbeel]]、[[wiki/entities/uc-berkeley]]
- 基线 / 对比：[[wiki/methods/ncsn]]、[[wiki/methods/diffusion-2015]]（上游对照）
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
> `ingest raw/literature-notes/hoDenoisingDiffusionProbabilistic2020.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-11T17:08:39.454+08:00 %%
