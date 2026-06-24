---
type: literature-note
citekey: zhaoEGSDEUnpairedImagetoImage2022
title: "EGSDE: Unpaired Image-to-Image Translation via Energy-Guided Stochastic Differential Equations"
aliases: ["@zhaoEGSDEUnpairedImagetoImage2022"]
authors: "Min Zhao, Fan Bao, Chongxuan Li, Jun Zhu"
firstAuthor: "Zhao"
year: 2022
itemType: preprint
doi: "10.48550/arXiv.2207.06635"
url: "http://arxiv.org/abs/2207.06635"
zotero: "zotero://select/library/items/YEE45LWP"
tags: [literature, computer-science---computer-vision-and-pattern-recognition]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-06-21
updated: 2026-06-21
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]"
---

# EGSDE: Unpaired Image-to-Image Translation via Energy-Guided Stochastic Differential Equations

> [!info] @zhaoEGSDEUnpairedImagetoImage2022 · Zhao et al. · 2022
> [Open in Zotero](zotero://select/library/items/YEE45LWP) · [DOI](https://doi.org/10.48550/arXiv.2207.06635) · [URL](http://arxiv.org/abs/2207.06635) · [PDF](file:///home/lonper/Zotero/storage/98SGHEP7/Zhao%20等%20-%202022%20-%20EGSDE%20Unpaired%20Image-to-Image%20Translation%20via%20Energy-Guided%20Stochastic%20Differential%20Equations.pdf)

## Abstract

> [!abstract]- Click to expand
> Score-based diffusion models (SBDMs) have achieved the SOTA FID results in unpaired image-to-image translation (I2I). However, we notice that existing methods totally ignore the training data in the source domain, leading to sub-optimal solutions for unpaired I2I. To this end, we propose energy-guided stochastic differential equations (EGSDE) that employs an energy function pretrained on both the source and target domains to guide the inference process of a pretrained SDE for realistic and faithful unpaired I2I. Building upon two feature extractors, we carefully design the energy function such that it encourages the transferred image to preserve the domain-independent features and discard domain-specific ones. Further, we provide an alternative explanation of the EGSDE as a product of experts, where each of the three experts (corresponding to the SDE and two feature extractors) solely contributes to faithfulness or realism. Empirically, we compare EGSDE to a large family of baselines on three widely-adopted unpaired I2I tasks under four metrics. EGSDE not only consistently outperforms existing SBDMs-based methods in almost all settings but also achieves the SOTA realism results without harming the faithful performance. Furthermore, EGSDE allows for flexible trade-offs between realism and faithfulness and we improve the realism results further (e.g., FID of 51.04 in Cat to Dog and FID of 50.43 in Wild to Dog on AFHQ) by tuning hyper-parameters. The code is available at https://github.com/ML-GSAI/EGSDE.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->


%% begin why-read %%

- 我想理解 EGSDE 如何把外部 energy function 接入 Score SDE / diffusion sampling，尤其是它如何把源图像条件转化为反向 SDE 中的梯度引导项。
    
- 这篇吸引我的点是：它不是重新训练一个 paired conditional diffusion model，而是在只训练目标域 score model 的基础上，用源域和目标域共同训练 / 构造的 energy function 做非配对 I2I。
    
- 这篇和 [[wiki/concepts/score-sde]]、[[wiki/methods/sdedit]]、[[wiki/concepts/classifier-guidance]]、[[wiki/concepts/energy-guidance]]、[[wiki/methods/ddbm]]、[[wiki/methods/dbim]]、[[wiki/methods/flux-kontext]]、[[wiki/benchmarks/afhq|AFHQ / unpaired I2I]] 相关（ILVR 待建页）。尤其想借它思考：如何在 diffusion / SDE / Flow 采样中，从 noisy state 得到关于 clean image 的判别式评分，并转化为 guidance。  

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
### Imported 2026-06-21 18:41

- 🟡 **p.1** — Building upon two feature extractors, we carefully design the energy function such that it encourages the transferred image to preserve the domain-independent features and discard domain-specific ones [⤴](zotero://open-pdf/library/items/98SGHEP7?page=1&annotation=LYRJPUCL)

- 🟡 **p.1** — product of experts [⤴](zotero://open-pdf/library/items/98SGHEP7?page=1&annotation=PNGWFVW7)

- 🟡 **p.2** — EGSDE defines a valid conditional distribution via a reverse time SDE that composites the energy function and the pretrained SDE. [⤴](zotero://open-pdf/library/items/98SGHEP7?page=2&annotation=KDETWLUX)

- 🟡 **p.3** — the training data in the source domain [⤴](zotero://open-pdf/library/items/98SGHEP7?page=3&annotation=I9G5MGGM)

- 🟡 **p.3** — dy = [f (y, t) − g(t)2(s(y, t) − ∇yE(y, x0, t))]dt + g(t)dw, (6) [⤴](zotero://open-pdf/library/items/98SGHEP7?page=3&annotation=CCLT3PY3)

- 🟡 **p.4** — The start point yM is sampled from the perturbation distribution qM|0(yM |x0) [32], where M = 0.5T typically. [⤴](zotero://open-pdf/library/items/98SGHEP7?page=4&annotation=UHVZYZUG)

- 🟡 **p.4** — solely in the target domain [⤴](zotero://open-pdf/library/items/98SGHEP7?page=4&annotation=MVKD6GR3)

- 🟡 **p.4** — although many other possibilities exist, [⤴](zotero://open-pdf/library/items/98SGHEP7?page=4&annotation=YPEUS2CZ)

- 🟡 **p.4** — = λsEqt|0(xt|x)Ss(y, xt, t) − λiEqt|0(xt|x)Si(y, xt, t), (8) [⤴](zotero://open-pdf/library/items/98SGHEP7?page=4&annotation=4AHKIN7Q)
  - 💬 *我的批注*：它避免了把 noisy y_t​ 直接和 clean x_0 比较，因为二者噪声水平不同，特征相似性可能没有意义。

- 🟡 **p.4** — In particular, Es(·, ·) is the all but the last layer of a classifier that is trained on both domains to predict whether an image is from the source domain or the target domain. [⤴](zotero://open-pdf/library/items/98SGHEP7?page=4&annotation=JTISZFUZ)

- 🟡 **p.5** — Ss(y, xt, t) = 1  HW  ∑  h,w  Eshw(xt, t)>Eshw(y, t)  ||Eshw(xt, t)||2 ||Eshw(y, t)||2  , (9) [⤴](zotero://open-pdf/library/items/98SGHEP7?page=5&annotation=9UZU8LNI)

- 🟡 **p.5** — a low-pass filter. [⤴](zotero://open-pdf/library/items/98SGHEP7?page=5&annotation=8U9FHNK5)

- 🟡 **p.5** — Si(y, xt, t) = −||Ei(y, t) − Ei(xt, t)||2 [⤴](zotero://open-pdf/library/items/98SGHEP7?page=5&annotation=ZTFS4TF9)

- 🟡 **p.6** — p ̃(yt|ys) ≈ N (μ(ys, h) − Σ(s, h)∇y′ E(y′, x0, t)|y′=μ(ys,h), Σ(s, h)I). (15) [⤴](zotero://open-pdf/library/items/98SGHEP7?page=6&annotation=QV5ZNGHY)

- 🟡 **p.9** — employ a low-pass filter as the domain-independent feature extractor [⤴](zotero://open-pdf/library/items/98SGHEP7?page=9&annotation=TRVIUUY3)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%


1. EGSDE 的核心方法是：只在目标域训练一个 score-based diffusion model，然后在反向 SDE 采样时加入源图像相关的 energy guidance：
    
    $$
	s_{\mathrm{EGSDE}}(y_t,t,x_0)=s_{\mathcal Y}(y_t,t)-\nabla_{y_t}\mathcal E(y_t,x_0,t).
	$$
    
    这里目标域 score 负责生成目标域真实图像，energy function 负责保留源图像的域无关内容并去掉源域特定内容。
    
1. EGSDE 的 energy 由两个专家组成： 
    $$  
    \mathcal E =
    \lambda_s\mathcal E_s  
    +  
    \lambda_i\mathcal E_i.  
    $$
    $\mathcal E_s$来自一个源域 / 目标域分类器的中间特征，用来降低生成样本和源图像在 domain-specific features 上的相似度，从而提升 realism；$\mathcal E_i$ 使用低通滤波器保留整体结构、颜色、背景等 domain-independent features，从而提升 faithfulness。
2. 这篇的关键实现不是先从 $y_t$ 估计 $\hat y_0$ 再评分，而是把源图像 $x_0$ 加噪到同一时间得到 $x_t$，然后比较 noisy sample $y_t$ 和 noisy source $x_t$：
    
    $$  
    x_0\to x_t,  
    \qquad  
    (y_t,x_t,t)\to \mathcal E\to\nabla_{y_t}\mathcal E.  
    $$
    
    因此 EGSDE 是一种 noisy-aligned energy guidance，而不是 clean-estimate-level guidance。
    
3. EGSDE 可以解释为 product of experts：
    
    $$  
    \tilde p(y_t\mid x_0)  
    \propto  
    p_{r1}(y_t\mid x_0)  
    p_{r2}(y_t\mid x_0)  
    p_f(y_t\mid x_0).  
    $$
    
    其中 $p_{r1}$ 是目标域 SDE / SDEdit realism expert，$p_{r2}$ 是丢弃源域特定特征的 realism expert，$p_f$ 是保留域无关特征的 faithful expert。这个解释说明了为什么 energy gradient 会作为反向 SDE drift correction 出现。
    
4. 对我的研究问题来说，EGSDE 的价值在于提供了一个已验证有效的 energy-guided SDE 框架；但它的局限是 faithful expert 只是低通滤波器，而且 energy 作用在 noisy state 上。自然的后续方向是把
    
    $$  
    \mathcal E(y_t,x_t,t)  
    $$
    
    替换成更强的 clean-estimate-level energy：
    
    $$  
    \widetilde{\mathcal E}(y_t,x_0,t) = E(\hat y_0(y_t,t),x_0),  
    $$
    
    并研究如何高效稳定地计算
    
    $$  
    \nabla_{y_t}E(\hat y_0(y_t,t),x_0) = \left(\frac{\partial \hat y_0}{\partial y_t}\right)^\top  
    \nabla_{\hat y_0}E.  
    $$
    
    这个思路可以迁移到 latent diffusion、Flow Matching、Rectified Flow 和 FLUX Kontext 中。  

%% end my-summary %%


## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/energy-guidance]]、[[wiki/concepts/classifier-guidance]]、[[wiki/concepts/score-sde]]
- 方法：[[wiki/methods/egsde]]、[[wiki/methods/sdedit]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/jun-zhu]]、[[wiki/entities/tsinghua-university]]
- 基线 / 对比：[[wiki/benchmarks/afhq]]（Cat→Dog / Wild→Dog / Male→Female）；ILVR、CycleGAN 系（待 ingest）
- 本篇 wiki source 页：[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
-
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/zhaoEGSDEUnpairedImagetoImage2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/zhaoEGSDEUnpairedImagetoImage2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-21T18:41:47.414+08:00 %%
