---
type: literature-note
citekey: rombachHighResolutionImageSynthesis2022
title: High-Resolution Image Synthesis with Latent Diffusion Models
aliases:
  - "@rombachHighResolutionImageSynthesis2022"
authors: Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, Björn Ommer
firstAuthor: Rombach
year: 2022
itemType: preprint
doi: 10.48550/arXiv.2112.10752
url: http://arxiv.org/abs/2112.10752
zotero: zotero://select/library/items/96P3CK9W
tags:
  - literature
  - todo
status: read
priority: P2
my-rating: "4"
created: 2026-05-27
updated: 2026-05-27
ingested_to_wiki: true
wiki_page: "[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"
---

# High-Resolution Image Synthesis with Latent Diffusion Models

> [!info] @rombachHighResolutionImageSynthesis2022 · Rombach et al. · 2022
> [Open in Zotero](zotero://select/library/items/96P3CK9W) · [DOI](https://doi.org/10.48550/arXiv.2112.10752) · [URL](http://arxiv.org/abs/2112.10752) · [PDF](file:///home/lonper/Zotero/storage/TRWDFKPK/Rombach%20等%20-%202022%20-%20High-Resolution%20Image%20Synthesis%20with%20Latent%20Diffusion%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> By decomposing the image formation process into a sequential application of denoising autoencoders, diffusion models (DMs) achieve state-of-the-art synthesis results on image data and beyond. Additionally, their formulation allows for a guiding mechanism to control the image generation process without retraining. However, since these models typically operate directly in pixel space, optimization of powerful DMs often consumes hundreds of GPU days and inference is expensive due to sequential evaluations. To enable DM training on limited computational resources while retaining their quality and flexibility, we apply them in the latent space of powerful pretrained autoencoders. In contrast to previous work, training diffusion models on such a representation allows for the first time to reach a near-optimal point between complexity reduction and detail preservation, greatly boosting visual fidelity. By introducing cross-attention layers into the model architecture, we turn diffusion models into powerful and flexible generators for general conditioning inputs such as text or bounding boxes and high-resolution synthesis becomes possible in a convolutional manner. Our latent diffusion models (LDMs) achieve a new state of the art for image inpainting and highly competitive performance on various tasks, including unconditional image generation, semantic scene synthesis, and super-resolution, while significantly reducing computational requirements compared to pixel-based DMs. Code is available at https://github.com/CompVis/latent-diffusion .

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 想理解为什么现代高分辨率图像生成不直接在 pixel space 做 diffusion，而是转到 latent space。
- 想理解 cross-attention 如何把 text / semantic map / layout 等条件注入 U-Net，进而统一 text-to-image、layout-to-image、super-resolution、inpainting 等任务。
- 想搞清楚 perceptual compression、semantic compression、autoencoder、VQGAN/VAE/KL-reg/VQ-reg 之间的关系。
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
### Imported 2026-05-27 10:43

- 🟡 **p.1** — To enable DM training on limited computational resources while retaining their quality and flexibility, we apply them in the latent space of powerful pretrained autoencoders. [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=1&annotation=WALITQFQ)

- 🟡 **p.1** — By introducing cross-attention layers into the model architecture, we turn diffusion models into powerful and flexible generators for general conditioning inputs such as text or bounding boxes and high-resolution synthesis becomes possible in a convolutional manner. [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=1&annotation=72PINVAV)

- 🟡 **p.2** — Departure to Latent Space [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=2&annotation=R8HMJIRV)

- 🟡 **p.2** — (v) Moreover, we design a general-purpose conditioning mechanism based on cross-attention, enabling multi-modal training. We use it to train class-conditional, text-to-image and layout-to-image models. [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=2&annotation=CQKKBGCY)

- ⚫ **p.3** — Such an approach offers several advantages [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=3&annotation=ZE32YL2S)
  - 💬 *我的批注*：这种几个优势的反复说的没有良心

- 🟡 **p.4** — a conditional denoising autoencoder θ(zt, t, y) [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=4&annotation=ZTFW5A4P)
  - 💬 *我的批注*：score的贝叶斯公式之外的另一种方法

- 🟡 **p.4** — Attention(Q, K, V ) = softmax  ( QKT  √d  )  · V , with  Q = W (i)  Q · φi(zt), K = W (i)  K · τθ(y), V = W (i)  V · τθ(y). [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=4&annotation=LEQG9HFF)

- 🟡 **p.5** — we then learn the conditional LDM via  LLDM := EE(x),y, ∼N (0,1),t  [  ‖ − θ(zt, t, τθ(y))‖2  2  ]  , (3) [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=5&annotation=PUIN94F7)

- 🟡 **p.5** — transformers [97] when y are text prompts ( [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=5&annotation=YRI758YH)
  - 💬 *我的批注*：for embedding

- 🟡 **p.7** — LDM-KL-8-G [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=7&annotation=WFCI3QEY)

### Imported 2026-05-27 15:41

- 🟡 **p.7** — 4.3.2 Convolutional Sampling Beyond 2562 [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=7&annotation=L7MRTJLF)
  - 💬 *我的批注*：LDM 的 latent diffusion 不仅能在固定分辨率工作，还能借助卷积结构扩展到更高分辨率。使用非token型空间对齐条件

- 🟡 **p.7** — We use this to train models for semantic synthesis, super-resolution (Sec. 4.4) and inpainting (Sec. 4.5). [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=7&annotation=TUEJFJ5L)

- 🟡 **p.7** — (ii) a rescaled version, scaled by the component-wise standard deviation. [⤴](zotero://open-pdf/library/items/TRWDFKPK?page=7&annotation=ECLQKJK4)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. LDM 的核心不是改写 DDPM 数学，而是改变 diffusion 发生的空间：先用感知自编码器把图像 $x$ 编码成二维 latent $z=\mathcal E(x)$，再在$z$ 上训练标准噪声预测扩散模型，最后用 decoder $\mathcal D$ 解码回图像。  
2. autoencoder 负责 perceptual compression：用 reconstruction/perceptual/adversarial loss 保留视觉上重要的信息，去掉像素空间中大量感知无关的高频细节；diffusion 负责在这个更低维、更适合建模的 latent space 中学习生成分布。  
3. 下采样因子 $f$ 是 LDM 的关键 trade-off：$f$ 太小接近 pixel diffusion，训练慢；$f$ 太大信息损失严重，质量上界下降；论文实验中 LDM-4 / LDM-8 在效率和感知质量之间最好。  
4. 条件 LDM 统一写成 $\epsilon_\theta(z_t,t,\tau_\theta(y))$：文本/layout 等 token 条件通过 cross-attention 注入 U-Net，其中 $Q$ 来自 U-Net latent feature，$K,V$ 来自条件编码器；空间对齐条件如 semantic map、低清图、mask 则可以直接 concat 到 noisy latent。  
5. LDM 是 Stable Diffusion 这类系统的直接基础：latent diffusion + VAE/autoencoder + U-Net + cross-attention + classifier-free guidance。它显著降低训练/采样成本，但仍有多步采样慢于 GAN、像素级精确任务受 autoencoder 重建上限限制等问题。  
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
-
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
- [ ]
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/rombachHighResolutionImageSynthesis2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/rombachHighResolutionImageSynthesis2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-27T15:41:08.318+08:00 %%
