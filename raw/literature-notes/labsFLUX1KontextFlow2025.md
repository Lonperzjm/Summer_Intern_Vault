---
type: literature-note
citekey: labsFLUX1KontextFlow2025
title: "FLUX.1 Kontext: Flow Matching for In-Context Image Generation and Editing in Latent Space"
aliases: ["@labsFLUX1KontextFlow2025"]
authors: "Black Forest Labs, Stephen Batifol, Andreas Blattmann, Frederic Boesel, Saksham Consul, Cyril Diagne, Tim Dockhorn, Jack English, Zion English, Patrick Esser, Sumith Kulal, Kyle Lacey, Yam Levi, Cheng Li, Dominik Lorenz, Jonas Müller, Dustin Podell, Robin Rombach, Harry Saini, Axel Sauer, Luke Smith"
firstAuthor: "Labs"
year: 2025
itemType: preprint
doi: "10.48550/arXiv.2506.15742"
url: "http://arxiv.org/abs/2506.15742"
zotero: "zotero://select/library/items/EYI5N7UE"
tags: [literature, todo]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-06-02
updated: 2026-06-02
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/labsFLUX1KontextFlow2025]]"
---

# FLUX.1 Kontext: Flow Matching for In-Context Image Generation and Editing in Latent Space

> [!info] @labsFLUX1KontextFlow2025 · Labs et al. · 2025
> [Open in Zotero](zotero://select/library/items/EYI5N7UE) · [DOI](https://doi.org/10.48550/arXiv.2506.15742) · [URL](http://arxiv.org/abs/2506.15742) · [PDF](file:///home/lonper/Zotero/storage/T2T3VZXA/Labs%20等%20-%202025%20-%20FLUX.1%20Kontext%20Flow%20Matching%20for%20In-Context%20Image%20Generation%20and%20Editing%20in%20Latent%20Space.pdf)

## Abstract

> [!abstract]- Click to expand
> We present evaluation results for FLUX.1 Kontext, a generative flow matching model that unifies image generation and editing. The model generates novel output views by incorporating semantic context from text and image inputs. Using a simple sequence concatenation approach, FLUX.1 Kontext handles both local editing and generative in-context tasks within a single unified architecture. Compared to current editing models that exhibit degradation in character consistency and stability across multiple turns, we observe that FLUX.1 Kontext improved preservation of objects and characters, leading to greater robustness in iterative workflows. The model achieves competitive performance with current state-of-the-art systems while delivering significantly faster generation times, enabling interactive applications and rapid prototyping workflows. To validate these improvements, we introduce KontextBench, a comprehensive benchmark with 1026 image-prompt pairs covering five task categories: local editing, global editing, character reference, style reference and text editing. Detailed evaluations show the superior performance of FLUX.1 Kontext in terms of both single-turn quality and multi-turn consistency, setting new standards for unified image processing models.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 我想搞清楚 FLUX.1 Kontext 如何把 Flow Matching 真正用到统一的图像生成与编辑里，尤其是它怎样把“文本条件 + 参考图像 / 源图像”统一表示为同一个上下文输入，而不是走传统的图像到图像加噪或额外控制分支路线。
- 我尤其想确认：它本质上是否可以理解为“潜空间中的条件 Rectified Flow / Flow Matching 模型”，并进一步理解它与 Stable Diffusion img2img、ControlNet、InstructPix2Pix、DDBM / DBIM 这些方法在机制上到底有什么根本差异。
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
### Imported 2026-06-02 17:22

- 🟡 **p.1** — a generative flow matching model [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=1&annotation=D56I8EYD)

- 🟡 **p.1** — unifies image generation and editing [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=1&annotation=IYBKJ6NT)

- 🟡 **p.1** — incorporating semantic context from text and image inputs. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=1&annotation=U8UW5TRT)

- 🟡 **p.1** — KontextBench, a comprehensive benchmark with 1026 image-prompt pairs covering five task categories: [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=1&annotation=BVRKWTES)

- 🟡 **p.1** — Local editing. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=1&annotation=8BKHJWBK)

- 🟡 **p.3** — Generative editing. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=3&annotation=IZEZ5EC9)

- 🟡 **p.4** — on a concatenated sequence of context and instruction tokens. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=4UWUQQ39)

- 🟡 **p.4** — Character consistency [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=BPXW425H)

- 🟡 **p.4** — Interactive speed [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=P8DYAWQQ)

- 🟡 **p.4** — Iterative application [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=DMG5DFX9)

- 🟡 **p.4** — 2 FLUX.1 [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=FI7RW33P)
  - 💬 *我的批注*：FLUX.1 Kontext=FLUX.1 基础模型+上下文图像与指令输入格式+统一生成和编辑训练

- 🟡 **p.4** — Figure 3: A fused DiT block equipped with rotary positional embeddings [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=JF3K62MY)
  - 💬 *我的批注*：你刚才看的图就是单流 DiT 模块。

- 🟡 **p.4** — By scaling up the training compute and using 16 latent channels, [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=3IHR9Z7S)

- 🟡 **p.4** — a mix of double stream and single stream [38] blocks [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=GB6NTB5N)

- 🟡 **p.4** — We utilize factorized three–dimensional Rotary Positional Embeddings (3D RoPE) [53]. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=QYRRE7XA)

- 🟡 **p.4** — p(x | y, c) (1) [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=4&annotation=MT5NPGNQ)

- 🟡 **p.5** — collect and curate millions of relational pairs (x | y, c) for optimization. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=5&annotation=SFHKGIEW)

- 🟡 **p.5** — instead encode them into a token sequence [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=5&annotation=E7BK45KB)

- 🟡 **p.5** — Token sequence construction. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=5&annotation=EB3IIBG3)

- 🟡 **p.5** — t u = (t, h, w), then we set ux = (0, h, w) for the target tokens [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=5&annotation=LLEQ4UHV)

- 🟡 **p.5** — Rectified-flow objective. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=5&annotation=HB8GV2H9)

- 🟡 **p.6** — Adversarial Diffusion Distillation [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=6&annotation=2WBS8PLG)

- 🟡 **p.6** — latent adversarial diffusion distillation [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=6&annotation=3TQELS5P)

- 🟡 **p.6** — single context images [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=6&annotation=XDSZ3R52)

- 🟡 **p.7** — Flash Attention 3 [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=7&annotation=MN87LE5Y)

- 🟡 **p.7** — introduce KontextBench [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=7&annotation=XLCABNA7)

- 🟡 **p.7** — 4.1 KontextBench – Crowd-sourced Real-World Benchmark for In-Context Tasks [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=7&annotation=G2KXPHXG)

- 🟡 **p.7** — It spans five core tasks: local instruction editing (416 examples), global instruction editing (262), text editing (92), style reference (63), and character reference (193). [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=7&annotation=T6IC45UB)

- 🟡 **p.8** — Image-to-Image Results. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=8&annotation=TYS26KMS)
  - 💬 *我的批注*：图像质量+局部编辑+角色参考+风格参考+文本编辑+计算效率.

- 🟡 **p.8** — Text-to-Image Results. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=8&annotation=8EJF6A9N)
  - 💬 *我的批注*：提示词遵循+美学+真实感+排版准确性+推理速度.

- 🟡 **p.9** — 4.3 Iterative Workflows [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=9&annotation=7HW9SWNW)

- 🟡 **p.12** — KontextBench: A real-world benchmark with 1 026 imageprompt pairs. [⤴](zotero://open-pdf/library/items/T2T3VZXA?page=12&annotation=BP4VKC44)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. FLUX.1 Kontext 的核心建模对象是条件分布
   p_\theta(x \mid y, c),
   其中 x 是目标图像，y 是可选的上下文图像，c 是文本提示或编辑指令。于是 y=\varnothing 时它就是文本到图像，y\neq\varnothing 时它就是图像编辑、参考生成或图像到图像；它的统一点在于把这些任务都视为“上下文条件生成”。

2. 它本质上是一个建立在 FLUX.1 之上的潜空间 Rectified Flow Transformer。图像先通过自编码器编码到潜空间，再在潜空间里做 Flow Matching；训练目标是速度预测而不是噪声预测、score 预测或 bridge score 预测。它学习的是
   v_\theta(z_t,t,y,c)\approx (\varepsilon-x),
   其中
   z_t=(1-t)x+t\varepsilon。
   所以它是条件 Flow Matching，不是 DDBM / DBIM 那种显式 source-target bridge。

3. Kontext 最关键的设计是“上下文图像作为词元进入模型”。上下文图像不会像 SDEdit / img2img 那样被加噪后作为采样起点，也不是像 ControlNet 那样走额外控制分支；它是先被编码成潜变量词元，再与目标图像词元做序列拼接，然后与文本词元一起通过双流 / 单流 Transformer 做统一注意力建模。这使它可以自然支持局部编辑、全局编辑、角色参考、风格参考、文本编辑和多轮编辑。

4. 从知识图谱上看，FLUX.1 Kontext 最准确的定位是：
   “LDM 风格的潜空间生成框架 + Rectified Flow / Flow Matching 训练目标 + 多模态 in-context conditioning”。
   它和 Stable Diffusion img2img 的本质区别是：后者主要靠“从哪里开始反推”，而 Kontext 更像是“把图像和文本都当作上下文，让模型学习每一步该往哪里流”。因此它更适合统一参考生成、多轮编辑和身份保持，而不是单纯做一次性图像重绘。

5. 论文最有启发的地方有两点。第一，很多图像编辑问题不一定要建模成显式 bridge，也可以建模成条件生成：
   p_\theta(x\mid y,c)。
   第二，把参考图像、源图像、视觉标记、文本指令都统一成上下文，让模型在注意力里自行学习“如何利用上下文”，这比针对每个任务单独设计一套结构更有通用性。实验上它最强的点是角色一致性、多轮编辑稳定性、局部编辑与文本编辑，同时速度也明显快，说明这种统一的上下文式生成模型很可能是未来图像编辑系统的重要方向。
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：
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
> `ingest raw/literature-notes/labsFLUX1KontextFlow2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/labsFLUX1KontextFlow2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-02T17:22:50.263+08:00 %%
