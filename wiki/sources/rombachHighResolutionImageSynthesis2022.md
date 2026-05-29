---
type: source
title: "High-Resolution Image Synthesis with Latent Diffusion Models"
aliases: [LDM, Latent Diffusion Models, "Rombach et al. 2022", Stable Diffusion 论文, SD 论文]
tags: [diffusion, latent-space, autoencoder, cross-attention, text-to-image, foundational]
status: stable
created: 2026-05-27
updated: 2026-05-28
raw: "[[raw/literature-notes/rombachHighResolutionImageSynthesis2022]]"
authors: [Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, Björn Ommer]
venue: CVPR 2022
year: 2022
arxiv: "2112.10752"
---

# High-Resolution Image Synthesis with Latent Diffusion Models

> 文献笔记：[[raw/literature-notes/rombachHighResolutionImageSynthesis2022]] · arXiv [2112.10752](http://arxiv.org/abs/2112.10752) · Rombach et al. 2022 (CVPR)
> 本页是 [[wiki/methods/ldm|LDM]] / Stable Diffusion 的方法论原点；几乎所有后续 text-guided editing 都默认建立在这条管线之上。

## 一句话

LDM **不重写 diffusion 数学**，而是把扩散搬到**预训练感知自编码器**给出的低维 latent $z=\mathcal E(x)$ 上：先用 $\mathcal E/\mathcal D$ 做一次「[[wiki/concepts/perceptual-compression|感知压缩]]」剥掉像素空间里大量感知无关的高频细节，再让 U-Net 在 $z$ 上学[[wiki/concepts/diffusion-process|扩散]]——训练/采样代价显著下降。条件通过 **[[wiki/concepts/cross-attention|cross-attention]]** 注入 U-Net（token 化条件如文本），或对**空间对齐条件**（semantic map / 低清图 / mask）直接 concat 到 noisy latent。这一管线 + KL-reg 的 $f=8$ 自编码器即 [[wiki/methods/ldm|LDM-KL-8]]，是 Stable Diffusion 的直接前身。

## Motivation

像素空间训 diffusion 太贵——DDPM 类 SOTA 模型动辄数百 GPU-day、采样上千次网络评估。但 likelihood-based diffusion 之所以贵，其实是因为它在**像素空间**里同时承担了两件事：

1. **perceptual compression**：去掉高频、纹理细节等感知无关信号；
2. **semantic compression**：建模语义/概念层面的分布。

作者的核心 motivation 是把这两件事**分阶段解耦**——先用 autoencoder 一次性完成第 (1) 步（一次性训练，可在多 task 复用），再让 diffusion 只在更紧凑、感知等价的 latent space 上做第 (2) 步。同时希望同一架构能处理**任意条件**（text、bounding box、layout、semantic map、低清图、mask…），所以引入 cross-attention 作为统一注入接口。

🟡 p.1：「we apply them in the latent space of powerful pretrained autoencoders」「By introducing cross-attention layers ... we turn diffusion models into powerful and flexible generators for general conditioning inputs such as text or bounding boxes」是论文的两条主张句。

## Method

### 0. 两阶段管线

**数据流**（像素 → latent → 扩散 → latent → 像素）：

$$
\underbrace{x}_{\text{像素}}\;\xrightarrow{\;\mathcal E\;}\;\underbrace{z}_{\text{latent}}\;\xrightarrow{+\varepsilon}\;z_t\;\xrightarrow{\;\varepsilon_\theta(z_t,t,\tau_\theta(y))\;}\;\hat z_0\;\xrightarrow{\;\mathcal D\;}\;\underbrace{\hat x}_{\text{像素}}
$$

- **阶段 1**（perceptual compression，**一次性预训练**，多任务复用）：自编码器 $\mathcal E,\mathcal D$ 用 [[wiki/concepts/perceptual-compression|reconstruction + perceptual (LPIPS) + adversarial loss]] 训练，**独立**于 diffusion。下采样因子 $f=H/h=W/w\in\{1,2,4,8,16,32\}$；编码后 $z\in\mathbb R^{h\times w\times c}$。两种正则：**KL-reg**（轻 KL 拉向 $\mathcal N(0,I)$，类 VAE）/ **VQ-reg**（VQ codebook，但 quantization 放进 decoder 内，diffusion 仍作用在连续 latent 上）。负责数据流前后两段 $x\!\to\!z$ 与 $\hat z_0\!\to\!\hat x$。
- **阶段 2**（semantic compression，**任务相关**）：在 $z$ 空间训 standard [[wiki/methods/ddpm|DDPM]]/[[wiki/methods/ddim|DDIM]] 式扩散——训练目标几乎照搬，仅把 $x$ 换成 $z$。U-Net + cross-attention 负责数据流中段 $z\!\to\!z_t\!\to\!\hat z_0$。

🟡 p.2 "Departure to Latent Space"：这一节是论文的关键论点——「**training diffusion models on such a representation allows for the first time to reach a near-optimal point between complexity reduction and detail preservation**」。

### 1. 训练目标

无条件 LDM：
$$L_{\mathrm{LDM}} = \mathbb E_{\mathcal E(x),\,\varepsilon\sim\mathcal N(0,I),\,t}\big[\|\varepsilon - \varepsilon_\theta(z_t, t)\|^2\big].$$
与 [[wiki/concepts/epsilon-parameterization|ε-prediction]] 完全相同的形式，只是 $x_t\to z_t=\sqrt{\bar\alpha_t}z+\sqrt{1-\bar\alpha_t}\varepsilon$。

条件 LDM（🟡 p.5 eq. 3）：
$$L_{\mathrm{LDM}} = \mathbb E_{\mathcal E(x),\,y,\,\varepsilon,\,t}\big[\|\varepsilon - \varepsilon_\theta(z_t, t, \tau_\theta(y))\|^2\big].$$
其中 $\tau_\theta$ 是把任意模态 $y$ 编码成 token 序列 $\tau_\theta(y)\in\mathbb R^{M\times d_\tau}$ 的模块（文本走 BERT-style transformer，🟡 p.5：「transformers when y are text prompts」）。

### 2. Cross-Attention：通用条件注入接口

把 $\tau_\theta(y)$ 注入 U-Net 中间层 $\phi_i(z_t)\in\mathbb R^{N\times d_\epsilon^i}$，🟡 p.4：

$$\mathrm{Attention}(Q,K,V)=\mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt d}\right)V,\quad
Q=W_Q^{(i)}\!\cdot\!\phi_i(z_t),\;\;K=W_K^{(i)}\!\cdot\!\tau_\theta(y),\;\;V=W_V^{(i)}\!\cdot\!\tau_\theta(y).$$

- **空间维度（来自 $z$）走 Q**，**条件 token 维度（来自 $y$）走 K/V**——天然支持「**每个空间位置自适应地从条件序列中检索相关 token**」，是 text-to-image 和后来一票 attention-injection 编辑方法（[[wiki/concepts/cross-attention|cross-attention]] 详见专页）的物理基础。
- ⚫ 用户批注（p.4）："**score 的贝叶斯公式之外的另一种方法**"——指条件 LDM 直接把 $y$ 作为网络输入而非通过引导项 $\nabla\log p(y\mid x)$ 后注入。这条评注精确：cross-attention 是 **conditional drift**，与 [[wiki/concepts/classifier-free-guidance|CFG]] 的**贝叶斯拆分式 guidance** 是**正交且通常组合使用**的两个机制（SD 实践即 conditional UNet + CFG 联用）。

### 3. Convolutional Sampling Beyond 256²（🟡 p.7 §4.3.2）

对**空间对齐**条件（semantic map、低清图、inpainting mask 等），$\tau_\theta(y)$ 不走 cross-attention，而是**直接 concat 到 noisy latent** 沿通道维。由于 U-Net 是全卷积，这种"非 token 型空间对齐条件"可以让 LDM **在比训练分辨率更高的 latent grid 上采样**——直接外推到 $512^2, 1024^2$ 及以上。用户批注："LDM 的 latent diffusion 不仅能在固定分辨率工作，还能借助卷积结构扩展到更高分辨率。使用非 token 型空间对齐条件"——精确。

### 4. 关键超参：下采样因子 $f$（trade-off 旋钮）

| $f$ | 行为 |
|---|---|
| $f=1$（无压缩） | 退化为 pixel diffusion，训练慢、采样 NFE 高 |
| $f=2,4$ | LDM-2 / LDM-4 |
| $f=8$（**甜点**） | **LDM-KL-8**，与 LDM-4 几乎并列最佳；后被 Stable Diffusion 采用 |
| $f=16, 32$ | autoencoder 重建上限拖累整体质量；细节丢失 |

🟡 p.7 用户高亮 "LDM-KL-8-G" —— 这正是 SD 的祖先：KL-reg 自编码器、$f=8$、用 [[wiki/concepts/classifier-free-guidance|guidance]]（"-G"）。论文 Fig 7 也展示了 $f$ 的非单调影响——太小训练慢，太大失真。

### 5. Latent 的归一化（🟡 p.7）

训练前对 $\mathcal E(x)$ 做"component-wise standard deviation rescaling"以稳定扩散训练数值条件——SD 的 `scaling_factor=0.18215` 即此处。

## Results

主要数字（论文 Table 1–6，挑相关）：

- **Class-conditional ImageNet 256²**：LDM-4-G（CFG）FID **3.60**，与 [[wiki/methods/ddpm|ADM]] (3.94) 持平甚至更好，但**训练 / 采样开销显著更低**（论文 Table 18：训练 GPU-day 数和采样 NFE 都比 ADM 少一个量级）。
- **CelebA-HQ 256² uncond**：LDM-4 FID **5.11**（当时 SOTA）。
- **Text-to-Image (LAION-400M 训练，COCO 零样本)**：LDM-KL-8-G CFG 后 FID **12.63**（论文 §4.3.1，与 GLIDE / DALL·E 同档而 compute 远低）。
- **Inpainting (Places)**：LDM-KL-8 在 LaMa / CoModGAN 之上拿到 SOTA。
- **Layout-to-Image / Semantic-to-Image / Super-Resolution (4×, ImageNet)**：同一管线、改 $\tau_\theta$ 或走 concat 通道即可，皆达到或接近 SOTA（论文 §4.3.2 / §4.4 / §4.5）。

> 论文 take-home：**同一 LDM 管线 + 同一 backbone**，仅靠 $\tau_\theta$ 与注入通道的切换（cross-attention vs concat），统一覆盖了 uncond / class-cond / text2img / layout2img / sem2img / SR / inpainting 等 7 类任务——这是"通用 generative backbone"的第一次干净示范。

## 关系（与已有 wiki 的关联）

- **直接前置（训练目标与公式）**：[[wiki/methods/ddpm|DDPM]] 的 ε-prediction 与 $L_\mathrm{simple}$ 几乎原样照搬，只是把 $x$ 换成 $z=\mathcal E(x)$；[[wiki/methods/ddim|DDIM]] 是 LDM 默认采样器之一。[[wiki/concepts/epsilon-parameterization|ε-prediction]]、[[wiki/concepts/diffusion-process]]、[[wiki/concepts/variational-bound-elbo]] 在 $z$ 空间继续适用。
- **本页直接催生的新页**：方法 [[wiki/methods/ldm]]；概念 [[wiki/concepts/cross-attention]]、[[wiki/concepts/perceptual-compression]]、[[wiki/concepts/latent-space-generative-modeling]]；实体 [[wiki/entities/robin-rombach]]、[[wiki/entities/bjorn-ommer]]、[[wiki/entities/compvis]]、[[wiki/entities/lmu-munich]]。
- **Guidance**：LDM-*-G 的 "-G" 即 [[wiki/concepts/classifier-free-guidance|CFG]]，与 cross-attention 注入**正交且通常组合**（cross-attention 改 conditional drift，CFG 在采样期把 conditional/unconditional ε 线性组合放大方向）。
- **与 [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]]**：LDM 没改 score / ε / 训练目标本身，所以 Song et al. 的 SDE/PF-ODE 视角在 $z$ 空间继续成立——SD 上跑 PF-ODE 采样器与 DDIM inversion 即此一致性的直接利用。
- **与 [[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] / [[wiki/sources/liuFlowStraightFast2022a|Rectified Flow]]**：LDM **不动训练目标**，FM/RF 才动。两者**正交**——SD3 / FLUX = **LDM 风格的 latent + autoencoder + cross-attention 注入** ⊕ **FM/RF 风格的速度场训练目标**。在 [[wiki/overview]] 「可变性光谱」语言下：LDM 在"组件维度"加了**压缩层**一档，FM/RF 在"训练目标"档施力——两条线汇合于 SD3 / FLUX。
- **下游编辑论文（皆默认 SD/LDM 底座）**：✅ [[wiki/methods/controlnet|ControlNet]]（已 ingest 2026-05-28）；待 ingest：Prompt-to-Prompt、Null-text Inversion、InstructPix2Pix、T2I-Adapter、IP-Adapter、Plug-and-Play、PnP-Diffusion、MasaCtrl、StyleAligned、RF-Inversion（这条还要求 RF 底座）… 这些方法的"在 latent 上做 inversion / attention injection / sideband 加性注入"全部默认 LDM 管线。
- 人物：[[wiki/entities/robin-rombach]]（一作）、[[wiki/entities/bjorn-ommer]]（通信作者 / 实验室 PI）；机构：[[wiki/entities/compvis]]、[[wiki/entities/lmu-munich]]。

## 对我的 thesis 的启示

按用户确认：将"autoencoder 重建上限会限制编辑保真度"作为 🟣 级 thesis-implication 处理。

- **给 overview「可变性光谱」补一档"压缩层"**：当前光谱（训练目标 → backbone → 采样器/guidance/介入时间步）是**沿"diffusion 算法本身"的维度**列开的；LDM 引入了一个**新维度**——"扩散发生前的信号被预处理到什么程度"。这一档独立于其他三档（LDM 不动训练目标、不动 backbone 类型、不动采样器），但对最终质量的天花板影响**直接且不可逆**——$\mathcal D(\mathcal E(x))\neq x$ 是 hard upper bound。
- **fidelity↔controllability trade-off 多了一个正交变量**：当前 working thesis 推论 2 把 fidelity/controllability 旋钮形式化为「在哪个 $t$ 注入 + 注入多强」（条件 score 引导项的逐 $t$ 量）。LDM 之上，**autoencoder 的 perceptual budget** 成为一个**正交的、更上游的 fidelity 上界**——即使在最优 $t$ 注入了最优强度的条件，若该编辑要求的"细节"已经被 $\mathcal E$ 丢弃（如细小文字、人脸 micro-feature、纹理高频），结果仍然失真。这是 thesis 当前框架未显式建模的一块。
- **对 inversion-based 编辑的影响**：所有 SD 上的 inversion（DDIM inversion、Null-text inversion、Negative-prompt inversion…）**都是在 latent 上做的往返**，pixel 空间从来不出现。这有两个推论：(i) inversion 的"reconstruction loss"评价指标本身就是 $\|\mathcal D(\hat z_0)-x\|$，已被 $\mathcal E/\mathcal D$ 的有损性污染；(ii) DDIM inversion 的失败模式（[[wiki/methods/ddim|DDIM]] 提到的）可能与 autoencoder 重建误差**纠缠**，不易解耦——这是 thesis 可立的一个诊断维度。
- **overview 主要派系→inversion-based 应加注**：所有该派系方法都隐含"在 latent 上往返"，应在 overview 该派系条目下注明此前提。
- **范式不变性的再次确认**：LDM 没动"迭代生成 + 网络预测噪声 + 沿生成链注入条件"的范式，只在范式之外（**外加一个一次性预处理层**）操作——给推论 1 的"范式 vs 组件"切分加一个干净佐证：**新组件可以加，但 paradigm 不动**。

> 拟据此**轻量更新** [[wiki/overview]]（可变性光谱新增"压缩层"档；主要派系→inversion-based 加 latent 前提注），不升 working thesis 版本号。已附 diff 到本次 ingest 的 commit 注释，由用户拍板是否进一步细化。

## Open questions / 待追

> 用户说明：本篇 annotation 没有 🔵 / 🟣 颜色标记（"太麻烦了全用的黄色"），所以以下问题由我从用户 my-summary、annotation 与 overview 接口处推断而出，**优先级以与 thesis 的接近度排序**。

- [ ] **P0**：autoencoder 重建上限在 text-guided **editing** 任务（而非生成）上的可量化影响——能否构造一个"绕过 autoencoder 直接重建"的对照来定量分离 $\mathcal E/\mathcal D$ 损失与扩散过程损失？这是 thesis 可立的一个干净实验。
- [ ] **P0**：DDIM inversion 在 LDM 上的失败模式是否与 autoencoder 重建误差耦合？做 ablation 看：去掉 $\mathcal E/\mathcal D$ 后的 pixel 空间 DDIM inversion 失败模式与 SD 上的失败模式是否同源。
- [ ] **P1**：cross-attention 注入与 CFG 在数学上的精确组合——CFG 的贝叶斯拆分式 + cross-attention 的 conditional drift，是否在"在哪个 $t$ 注入"维度上**有冲突**？SD 实践把它们当作可加但未必正确。
- [ ] **P1**：VQ-reg vs KL-reg 在编辑任务上是否有系统差异？论文给出生成 metrics 对比但**无编辑下游评测**；SD 选 KL-reg，VQGAN 系选 VQ-reg，差异在 editability 上是否显著？
- [ ] **P1**：SD3 / FLUX：LDM 压缩层 + RF/FM 训练目标的组合落地——SD3 paper（Esser et al. 2024）/ FLUX 仍未 ingest，是 overview「主要派系→flow-matching-based」的下一个直接落点。
- ✅ **P2 (2026-05-28 [[wiki/methods/controlnet|ControlNet]] ingest 关闭)**：layout-to-image / semantic-to-image 走 concat 通道而非 cross-attention，这条"空间对齐条件用 concat"的设计原则在后来的 ControlNet **未被保留**——ControlNet 改成"复制 SD encoder + middle 作 trainable copy + [[wiki/concepts/zero-convolution|zero conv]] 加性 residual 注入 12 条 skip + 1 个 middle"。两者的核心差异：(i) LDM concat 把条件直接拼通道，diffusion 仍要重训以学会用这个新通道；(ii) ControlNet sideband **冻结 SD 主干**，只训副本——避免灾难性遗忘、50k 数据即稳。详见 [[wiki/sources/zhangAddingConditionalControl2023]]。
