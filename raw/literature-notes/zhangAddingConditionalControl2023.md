---
type: literature-note
citekey: zhangAddingConditionalControl2023
title: "Adding Conditional Control to Text-to-Image Diffusion Models"
aliases: ["@zhangAddingConditionalControl2023"]
authors: "Lvmin Zhang, Anyi Rao, Maneesh Agrawala"
firstAuthor: "Zhang"
year: 2023
itemType: preprint
doi: "10.48550/arXiv.2302.05543"
url: "http://arxiv.org/abs/2302.05543"
zotero: "zotero://select/library/items/U5VHQ3JL"
tags: [literature, todo, poster]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-05-28
updated: 2026-05-28
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/zhangAddingConditionalControl2023]]"
---

# Adding Conditional Control to Text-to-Image Diffusion Models

> [!info] @zhangAddingConditionalControl2023 · Zhang et al. · 2023
> [Open in Zotero](zotero://select/library/items/U5VHQ3JL) · [DOI](https://doi.org/10.48550/arXiv.2302.05543) · [URL](http://arxiv.org/abs/2302.05543) · [PDF](file:///home/lonper/Zotero/storage/95G7SPNS/Zhang%20等%20-%202023%20-%20Adding%20Conditional%20Control%20to%20Text-to-Image%20Diffusion%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> We present ControlNet, a neural network architecture to add spatial conditioning controls to large, pretrained text-to-image diffusion models. ControlNet locks the production-ready large diffusion models, and reuses their deep and robust encoding layers pretrained with billions of images as a strong backbone to learn a diverse set of conditional controls. The neural architecture is connected with "zero convolutions" (zero-initialized convolution layers) that progressively grow the parameters from zero and ensure that no harmful noise could affect the finetuning. We test various conditioning controls, eg, edges, depth, segmentation, human pose, etc, with Stable Diffusion, using single or multiple conditions, with or without prompts. We show that the training of ControlNets is robust with small (<50k) and large (>1m) datasets. Extensive results show that ControlNet may facilitate wider applications to control image diffusion models.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 了解dataset的情况，了解controlnet作为挂件分开训练的方法。
- 想理解高校实验室在资源有限的情况下，如何围绕大模型做有意义的实验：不是从零训练开放域生成模型，而是在已有预训练模型上做微调、插件或条件控制。
- 想了解文生图 / 图像生成论文里的 dataset 使用方式：哪些任务需要大规模数据，哪些任务可以/必须用小中规模特定条件数据集完成。
- 想理解 ControlNet 作为 Stable Diffusion “外挂控制器”的训练范式：冻结强大的预训练主干，单独训练控制分支，从而低成本加入新能力。
- 想系统弄清 ControlNet 的结构细节：locked copy、trainable copy、zero convolution、条件图编码、skip connection 注入，以及这些设计为什么能稳定工作。
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
### Imported 2026-05-28 19:37

- 🟡 **p.1** — locks the productionready large diffusion model [⤴](zotero://open-pdf/library/items/95G7SPNS?page=1&annotation=MPN7SHKP)

- 🟡 **p.1** — “zero convolutions” [⤴](zotero://open-pdf/library/items/95G7SPNS?page=1&annotation=IDRDQ9G5)

- 🟡 **p.1** — using single or multiple conditions, [⤴](zotero://open-pdf/library/items/95G7SPNS?page=1&annotation=BU5R8W89)

- 🟡 **p.2** — Learning conditional controls for large text-to-image diffusion models in an end-to-end way is challenging. [⤴](zotero://open-pdf/library/items/95G7SPNS?page=2&annotation=IWNHGQ9Y)
  - 💬 *我的批注*：强控制需要学习，但大模型端到端学习成本高、风险大。

- 🟡 **p.2** — The direct finetuning or continued training of a large pretrained model with limited data may cause overfitting and catastrophic forgetting [31, 75]. [⤴](zotero://open-pdf/library/items/95G7SPNS?page=2&annotation=KYYM4LSY)

- 🟡 **p.2** — zero convolution layers, [⤴](zotero://open-pdf/library/items/95G7SPNS?page=2&annotation=GMBRF5NX)

- 🟡 **p.2** — (1) we propose ControlNet, a neural network architecture that can add spatially localized input conditions to a pretrained text-to-image diffusion model via efficient finetuning, [⤴](zotero://open-pdf/library/items/95G7SPNS?page=2&annotation=2SYK7MWH)

- 🟡 **p.2** — Adapter [⤴](zotero://open-pdf/library/items/95G7SPNS?page=2&annotation=H9P529ZP)

- 🟡 **p.3** — Zero-Initialized Layers are used by ControlNet for connecting network blocks. [⤴](zotero://open-pdf/library/items/95G7SPNS?page=3&annotation=QR6HTHYY)

- 🟡 **p.4** — clone the block to a trainable copy with parameters Θc [⤴](zotero://open-pdf/library/items/95G7SPNS?page=4&annotation=56WSSATP)

- 🔴 **p.5** — We observe that the model does not gradually learn the control conditions but abruptly succeeds in following the input conditioning image; usually in less than 10K optimization steps. As shown in Figure 4, we call this the “sudden convergence phenomenon”. [⤴](zotero://open-pdf/library/items/95G7SPNS?page=5&annotation=966ESPZU)

- 🟡 **p.6** — 4.2. Ablative Study [⤴](zotero://open-pdf/library/items/95G7SPNS?page=6&annotation=XHLCSLW9)
  - 💬 *我的批注*：1. 复杂空间条件需要深层 trainable copy，而不是浅层小 adapter。  
      2. zero convolution 是必要的

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 高校实验室做生成模型实验一般有两种现实路线：
    1. 在小/中数据集或窄域数据集上从零训练，用来验证新的建模思想、结构或训练目标；
    2. 使用已有大规模预训练模型做微调、adapter、LoRA、ControlNet 这类插件式方法，用较低成本研究新控制能力或新任务。
2. ControlNet 的核心不是改变 DDPM / LDM 的扩散数学，而是在 Stable Diffusion 的 latent U-Net 上加入空间条件控制。原模型仍然预测噪声：  
    $$ 
    \epsilon_\theta(z_t,t,c_t,c_f)  
    $$  
    训练目标仍然是噪声预测 MSE，只是额外加入了由条件图编码得到的 $c_f$。
    ![[zhangAddingConditionalControl2023-1779969426205.webp]]
3. ControlNet 的关键结构是：  
    $$ 
    \text{frozen Stable Diffusion U-Net}  
    +  
    \text{trainable encoder/middle copy}  
    +  
    \text{zero convolution residual injection}.  
    $$  
    它复制 Stable Diffusion 的 12 个 encoder blocks 和 1 个 middle block，输出加到原 U-Net 的 12 条 skip connections 和 1 个 middle block 上。![[zhangAddingConditionalControl2023-1779969462771.webp]]
    
4. zero convolution 是 ControlNet 稳定训练的核心。它是权重和 bias 初始化为 0 的 $1\times1$ 卷积，使 ControlNet 初始化时满足：  
    $$ 
    y_c=y.  
    $$  
    因此训练一开始模型等价于原 Stable Diffusion，不会被随机控制分支破坏；训练后 zero conv 逐渐学会注入控制残差。
5. ControlNet 之所以适合小数据训练，是因为它不需要重新学习自然图像分布。Stable Diffusion 负责保留图像生成能力和文本语义能力，ControlNet 只学习空间条件对已有去噪场的多尺度修正。因此它能用 edge、pose、depth、segmentation 等条件图实现强空间控制，并且比直接 fine-tune 更不容易灾难性遗忘。
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
> `ingest raw/literature-notes/zhangAddingConditionalControl2023.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/zhangAddingConditionalControl2023.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-28T19:37:59.810+08:00 %%
