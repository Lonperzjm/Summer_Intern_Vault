---
type: literature-note
citekey: mengSDEditGuidedImage2022
title: "SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations"
aliases: ["@mengSDEditGuidedImage2022"]
authors: "Chenlin Meng, Yutong He, Yang Song, Jiaming Song, Jiajun Wu, Jun-Yan Zhu, Stefano Ermon"
firstAuthor: "Meng"
year: 2022
itemType: preprint
doi: "10.48550/arXiv.2108.01073"
url: "http://arxiv.org/abs/2108.01073"
zotero: "zotero://select/library/items/K8TNH2EY"
tags: [literature, todo]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-05-29
updated: 2026-05-29
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/mengSDEditGuidedImage2022]]"
---

# SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations

> [!info] @mengSDEditGuidedImage2022 · Meng et al. · 2022
> [Open in Zotero](zotero://select/library/items/K8TNH2EY) · [DOI](https://doi.org/10.48550/arXiv.2108.01073) · [URL](http://arxiv.org/abs/2108.01073) · [PDF](file:///home/lonper/Zotero/storage/BEYTN39K/Meng%20等%20-%202022%20-%20SDEdit%20Guided%20Image%20Synthesis%20and%20Editing%20with%20Stochastic%20Differential%20Equations.pdf)

## Abstract

> [!abstract]- Click to expand
> Guided image synthesis enables everyday users to create and edit photo-realistic images with minimum effort. The key challenge is balancing faithfulness to the user input (e.g., hand-drawn colored strokes) and realism of the synthesized image. Existing GAN-based methods attempt to achieve such balance using either conditional GANs or GAN inversions, which are challenging and often require additional training data or loss functions for individual applications. To address these issues, we introduce a new image synthesis and editing method, Stochastic Differential Editing (SDEdit), based on a diffusion model generative prior, which synthesizes realistic images by iteratively denoising through a stochastic differential equation (SDE). Given an input image with user guide of any type, SDEdit first adds noise to the input, then subsequently denoises the resulting image through the SDE prior to increase its realism. SDEdit does not require task-specific training or inversions and can naturally achieve the balance between realism and faithfulness. SDEdit significantly outperforms state-of-the-art GAN-based methods by up to 98.09% on realism and 91.72% on overall satisfaction scores, according to a human perception study, on multiple tasks, including stroke-based image synthesis and editing as well as image compositing.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 我想把 SDEdit 放进 diffusion / Score SDE / img2img 的知识图谱里：它不是新的扩散理论，而是把预训练 score model 的反向生成过程从「纯噪声起点」改成「输入图像的中间加噪状态」。
- 我想理解为什么
  $$
  x_{\mathrm{input}}\rightarrow x_{t_0}\rightarrow x_0
  $$
  这件事可以实现 image editing：输入图像先被加噪，削弱不真实伪影；再由 reverse SDE 根据自然图像 score 拉回数据流形。
- 我想重点理解 $t_0$ / noise strength 的作用：它如何控制真实感和忠实性之间的权衡，以及它和后来的 DDPM/DDIM img2img、Stable Diffusion img2img 的 strength 参数是什么关系。
- 我也想弄清楚 SDEdit 和 GAN inversion、conditional GAN、ControlNet、inpainting、posterior sampling / bridge model 的区别：哪些方法需要训练或反演，哪些只是 sampling strategy。
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
### Imported 2026-05-29 18:33

- 🟡 **p.1** — Yang Song1 [⤴](zotero://open-pdf/library/items/BEYTN39K?page=1&annotation=XCJ6VWYA)

- 🟡 **p.1** — Jiaming Song1 [⤴](zotero://open-pdf/library/items/BEYTN39K?page=1&annotation=WVWIG5J7)

- 🟡 **p.1** — Stefano Ermon1 [⤴](zotero://open-pdf/library/items/BEYTN39K?page=1&annotation=XYTG29B2)

- 🟡 **p.2** — Given an input image with user guidance input, such as a stroke painting or an image with stroke edits, we can add a suitable amount of noise to smooth out undesirable artifacts and distortions (e.g., unnatural details at stroke pixels), while still preserving the overall structure of the input user guide. We then initialize the SDE with this noisy input, and progressively remove the noise to obtain a denoised result that is both realistic and faithful to the user guidance input [⤴](zotero://open-pdf/library/items/BEYTN39K?page=2&annotation=JSBE9ZU4)

- 🟡 **p.3** — Image synthesis with VE-SDE [⤴](zotero://open-pdf/library/items/BEYTN39K?page=3&annotation=RT4AJIIL)

- 🟡 **p.3** — x(t) = x(t + ∆t) + (σ2(t) − σ2(t + ∆t))sθ(x(t), t) + √σ2(t) − σ2(t + ∆t)z. (4) [⤴](zotero://open-pdf/library/items/BEYTN39K?page=3&annotation=T8CCMZWD)

- ⚫ **p.4** — Setup. The user provides a full resolution image x(g) in a form of manipulating RGB pixels, which we call a “guide”. The guide may contain different levels of details; a high-level guide contains only coarse colored strokes, a mid-level guide contains colored strokes on a real image, and a low-level guide contains image patches on a target image. We illustrate these guides in Fig. 1, which can be easily provided by non-experts. Our goal is to produce full resolution images with two desiderata: [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=5G35DPQD)

- ⚫ **p.4** — Realism. The image should appear realistic (e.g., measured by humans or neural networks). Faithfulness. The image should be similar to the guide x(g) (e.g., measured by L2 distance). [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=X5V35WJD)

- 🟡 **p.4** — we define the SDEdit procedure as follows:  Sample x(g)(t0) ∼ N (x(g); σ2(t0)I), then produce x(0) by iterating Equation 4. [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=KP67FC8I)

- 🟡 **p.4** — Essentially, SDEdit selects a particular time t0, add Gaussian noise of standard deviation σ2(t0) to the guide x(g) and then solves the corresponding reverse SDE at t = 0 to produce the synthesized x(0). [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=LC5ZN25K)

- 🟡 **p.4** — the key hyperparameter for SDEdit is t0, [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=KDH2ZV4Q)

- 🟡 **p.4** — From Fig. 3, we observe increased realism but decreased faithfulness as t0 increases. [⤴](zotero://open-pdf/library/items/BEYTN39K?page=4&annotation=6B6SV9Q5)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 直观图像，假设是加噪的$x_{in}$和$x_{target}$有大重合![[mengSDEditGuidedImage2022-1780042277363.webp]]
2. 核心算法![[mengSDEditGuidedImage2022-1780045307604.webp]]
3. SDEdit 的核心思想很简单：不是从纯噪声开始生成，而是先把用户输入图像加噪到某个中间时刻 $t_0$，再从这个 noisy guide 开始运行 reverse SDE：  
	$$  
	x^{(g)}  
	\rightarrow  
	x^{(g)}(t_0)  
	\rightarrow  
	x(0).  
	$$  
	在 VE-SDE 中：  
	$$
	x^{(g)}(t_0)=x^{(g)}+\sigma(t_0)z,\quad z\sim\mathcal N(0,I).  
	$$  
	在 VP/DDPM 中则是：  
	$$ 
	x_{t_0}=\alpha(t_0)x^{(g)}+\sigma(t_0)\epsilon.  
	$$  
	所以 SDEdit 本质上是「noisy initialization + reverse denoising」。

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
> `ingest raw/literature-notes/mengSDEditGuidedImage2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/mengSDEditGuidedImage2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-29T18:33:41.163+08:00 %%
