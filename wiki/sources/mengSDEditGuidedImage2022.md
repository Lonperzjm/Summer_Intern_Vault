---
type: source
title: "SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations"
aliases: [SDEdit, "Meng et al. 2022", "Stochastic Differential Editing", SDE editing]
tags: [diffusion, editing, score-sde, noising-based, inversion-free, training-free]
status: stable
created: 2026-05-29
updated: 2026-05-29
raw: "[[raw/literature-notes/mengSDEditGuidedImage2022]]"
authors: [Chenlin Meng, Yutong He, Yang Song, Jiaming Song, Jiajun Wu, Jun-Yan Zhu, Stefano Ermon]
venue: ICLR 2022
year: 2022
arxiv: "2108.01073"
---

# SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations

> 文献笔记：[[raw/literature-notes/mengSDEditGuidedImage2022]] · arXiv [2108.01073](http://arxiv.org/abs/2108.01073) · Meng et al. 2022 (ICLR) · 投稿 2021.08
> 本页是 [[wiki/overview]] 主要派系 **Inversion / noising-based** 类的奠基论文——也是该类里最朴素的一支：**不训练、不反演、不加 loss**，纯采样策略。

## 一句话

把任意预训练 [[wiki/concepts/score-sde|score SDE]] 模型的反向生成**起点**从"纯噪声"改成"用户 guide 图 $x^{(g)}$ 加噪到中间时刻 $t_0$"——加噪抹掉 guide 里的不真实伪影、reverse SDE 把它拉回自然图像流形。唯一旋钮 $t_0$（[[wiki/concepts/noising-strength|noising strength]]）控制 **realism↔faithfulness** 权衡，是 [[wiki/overview]] 推论 2「在 reverse 链哪个 $t$ 介入」假设迄今最早（2021.08）、最干净的经验实证，也是后来 **SD img2img 的 `strength` 参数**的理论原型。

## Motivation

guided image synthesis（如把手绘 stroke 变成真实照片）的核心矛盾是 **realism（看着真）↔ faithfulness（忠于输入）**。同期 GAN-based 路线两条都别扭：

- **conditional GAN**：每个任务要重训、要任务特定 loss；
- **GAN inversion**：把图反演回 latent 难、常需额外训练 / 优化。

🟡 p.2 用户高亮（论文核心句）：「we can **add a suitable amount of noise** to smooth out undesirable artifacts and distortions ... while still **preserving the overall structure** of the input user guide. We then initialize the SDE with this noisy input, and progressively remove the noise to obtain a denoised result that is **both realistic and faithful**」。

作者要的：一个**零训练、零反演、任务无关**的方法，只靠"加噪 + reverse"这件采样期操作就自然平衡 realism / faithfulness。

## Method

### 0. 直观：perturb-then-reverse

**原文 Fig 1（stroke painting → 加噪到分布重合 → reverse SDE → 真实塔楼）**：

![[mengSDEditGuidedImage2022-1780042277363.webp]]

上排是分布示意：guide $x^{(g)}$ 加噪后的分布与目标自然图加噪后的分布**有大重合**（用户 my-summary #1 的关键直觉）——这是 SDEdit 能 work 的前提。$t_0$ 要足够大让两分布重合（否则 reverse 拉不回真实流形），又要足够小让 guide 的结构信息不被完全抹掉。下排是像素实例：stroke 画 → 加噪成近似纯噪声 → reverse SDE 逐步去噪 → 真实塔楼。

### 1. 核心定义（🟡 p.4）

🟡 p.4 用户高亮（procedure 定义）：「Sample $x^{(g)}(t_0)\sim\mathcal N(x^{(g)};\sigma^2(t_0)I)$, then produce $x(0)$ by iterating Equation 4.」、「Essentially, SDEdit selects a particular time $t_0$, add Gaussian noise of standard deviation $\sigma^2(t_0)$ to the guide $x^{(g)}$ and then solves the corresponding reverse SDE at $t=0$」。

数据流：
$$
x^{(g)} \;\xrightarrow{\text{加噪到 } t_0}\; x^{(g)}(t_0) \;\xrightarrow{\text{reverse SDE}}\; x(0).
$$

- **VE-SDE**（用户 my-summary #3）：$x^{(g)}(t_0)=x^{(g)}+\sigma(t_0)z,\; z\sim\mathcal N(0,I)$；
- **VP / DDPM**：$x_{t_0}=\alpha(t_0)x^{(g)}+\sigma(t_0)\epsilon$。

### 2. Algorithm 1（VE-SDE，🟡 p.3 公式 (4)）

**原文 Algorithm 1**：

![[mengSDEditGuidedImage2022-1780045307604.webp]]

逐行：给定 guide $x^{(g)}$、$t_0$、去噪步数 $N$。先 $\Delta t = t_0/N$；采 $z\sim\mathcal N(0,I)$；**加噪初始化** $x\leftarrow x+\sigma(t_0)z$。然后 $n=N\to 1$ 迭代 reverse SDE（🟡 p.3 公式 (4)）：
$$
x \leftarrow x + \big(\sigma^2(t)-\sigma^2(t-\Delta t)\big)\,s_\theta(x,t) + \sqrt{\sigma^2(t)-\sigma^2(t-\Delta t)}\;z,
$$
其中 $t=t_0\cdot n/N$、每步重采 $z$。返回 $x$。

> 注意：第 0 步的"加噪初始化"只做**一次**（不像 [[wiki/methods/controlnet|ControlNet]] 在所有 $t$ 持续注入）；之后纯粹是标准 reverse SDE。这是 SDEdit "单次 $t_0$ 注入"与 ControlNet "全 $t$ sideband" 的本质区别（见关系节）。

### 3. 三级 guide（⚫ p.4 setup）

⚫ p.4 用户高亮：guide $x^{(g)}$ 是 full-resolution RGB 像素操作，按细节层级分三档：
- **high-level guide**：只有粗色块 stroke（stroke-based synthesis）；
- **mid-level guide**：真实图上加 stroke 编辑（stroke-based editing）；
- **low-level guide**：目标图上贴 image patch（image compositing）。

同一 SDEdit 流程覆盖这三类任务，只是 guide 的细节程度不同——$t_0$ 也要随之调（细节越多、越想忠实 → $t_0$ 越小）。

### 4. 两个 desiderata 的形式化（⚫ p.4）

⚫ p.4 用户高亮：
- **Realism**：图像看着真实（人评 / 神经网络度量）；
- **Faithfulness**：图像与 guide $x^{(g)}$ 相似（如 L2 距离度量）。

这两个量**天然冲突**，SDEdit 用单一 $t_0$ 在它们之间滑动——见下。

## Results

- **$t_0$ 是唯一关键超参，且单调 trade-off**（🟡 p.4）：「the key hyperparameter for SDEdit is $t_0$」、「From Fig. 3, we observe **increased realism but decreased faithfulness as $t_0$ increases**」。这条是 [[wiki/concepts/noising-strength|noising strength]] 旋钮的原始出处，也是推论 2 的直接经验证据。论文经验给出 $t_0\in[0.3, 0.6]$（归一化时间）多数任务最佳。
- **人评大幅超 GAN baselines**：realism 上最高 +98.09%、overall satisfaction 上 +91.72%（stroke-based synthesis / editing / compositing 多任务，对比 conditional GAN / GAN inversion 系）。
- **零训练 / 零反演**：任意预训练 score model 即插即用，无任务特定 loss——这是相对 GAN 路线的核心工程优势。
- **任务覆盖**：stroke-based image synthesis、stroke-based editing、image compositing（三级 guide 各对应一类）。

## 关系（与已有 wiki 的关联）

- **直接前置**：[[wiki/concepts/score-sde|Score SDE]]（Song et al. 2021）—— SDEdit 的 reverse SDE 公式 (4)、VE/VP-SDE 加噪式全部来自此；SDEdit 是 Score SDE 框架的一个**采样期应用**，不改任何理论。
- **不依附 Stable Diffusion**：SDEdit（2021.08）**早于** [[wiki/entities/stable-diffusion|SD]]（2022.08），用的是预训练 score model（论文在 LSUN / CelebA-HQ 等上的 pixel-space / score-SDE 模型）。**不要**把它误归为"SD 上的编辑方法"——它是底座无关的采样策略，可套在任何 score / diffusion prior 上（包括后来的 SD，即 SD img2img）。
- **本页直接催生的新页**：方法 [[wiki/methods/sdedit]]；概念 [[wiki/concepts/noising-strength]]（$t_0$ / `strength` 旋钮）；实体 [[wiki/entities/chenlin-meng]]、[[wiki/entities/jiajun-wu]]、[[wiki/entities/jun-yan-zhu]]、[[wiki/entities/yutong-he]]。
- **与 [[wiki/methods/ddim|DDIM]] / SD img2img**：SD img2img 的 `strength` 参数就是 $t_0$ 的归一化版——`strength=0.7` 即加噪到 70% 时间步再 reverse。SDEdit 是这条"img2img" 范式的理论原点（虽然 SD img2img 实现上常用 DDIM 而非 SDE）。
- **与 [[wiki/methods/controlnet|ControlNet]]（sideband 对照）**：两者都**不优化、不反演**，但注入方式正交——SDEdit 是**单次 $t_0$ 加噪初始化**（guide 信息进初始状态、之后纯 reverse）；ControlNet 是**全 $t$ 加性 sideband**（条件残差在每个去噪步注入）。这正是 thesis 要立的「全 $t$ sideband vs 单次/逐 $t$ 介入」对照的两个端点（见推论 2 caveat）。
- **与 [[wiki/concepts/sideband-conditioning|sideband 抽象]]**：SDEdit 的 sideband = "把 guide 加噪后当 reverse SDE 的**初始状态**"——是 [[wiki/overview]] inversion/noising-based 派系里**最朴素的一支**（连优化都不需要，对比 Null-text inversion 要优化空文本 embedding）。
- **作者网**：[[wiki/entities/chenlin-meng]]（一作，也是 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 二作）、[[wiki/entities/yutong-he]]、[[wiki/entities/yang-song]]、[[wiki/entities/jiaming-song]]、[[wiki/entities/jiajun-wu]]、[[wiki/entities/jun-yan-zhu]]、[[wiki/entities/stefano-ermon]]；机构 [[wiki/entities/stanford]]。

## 对我的 thesis 的启示

按用户确认：将"**SDEdit 的 $t_0$ 是推论 2「在 reverse 链哪个 $t$ 介入」假设的最早、最干净经验验证**"作为 🟣 级 thesis-implication 处理。

- **推论 2 拿到第一个直接编辑实证**：此前推论 2「高 $t$ 改结构、低 $t$ 改细节」只有 Score SDE 的形式化抓手（逐 $t$ 条件 score），缺 text-guided editing 论文的经验支撑。SDEdit 补上了——它的 Fig 3 实验**直接画出** $t_0$↑ ⇒ realism↑ / faithfulness↓ 的单调曲线，把"在哪个 $t$ 介入"从假设变成可测旋钮。这是 thesis fidelity↔controllability 章节的核心引用。
- **"介入时间步"与"介入强度"在 SDEdit 上合一**：注意 SDEdit 的 $t_0$ **同时**是"从哪个 $t$ 开始 reverse"和"加多少噪"——两个旋钮在 noising-based 编辑里耦合成一个。这与 [[wiki/concepts/classifier-free-guidance|CFG]] 的"逐 $t$ 引导强度 $w$"是**不同的旋钮**：CFG 在每个 $t$ 调强度、SDEdit 调起点。thesis 要厘清这两类旋钮的关系（可能正交、可组合）。
- **最朴素 sideband 的下界意义**：SDEdit 不优化、不训练，是 inversion/noising-based 派系成本最低的一支。它给 thesis 的 sideband 光谱一个**零成本端点**：Null-text inversion（优化几百步）、DDIM-inversion（确定性反演）、SDEdit（直接加噪）三者构成"忠实度 vs 成本"递增/递减光谱——这条光谱本身是 thesis 可立的一个分析框架。
- **realism↔faithfulness = fidelity↔controllability 的一个具体实例**：SDEdit 的两个 desiderata 给 thesis 的抽象 trade-off 一个可度量的落地（realism 用人评 / 神经网络，faithfulness 用 L2）——可作为 thesis 评测协议的参考。

> 拟据此**轻量更新** [[wiki/overview]]（inversion-based 派系改名 inversion/noising-based 并填 SDEdit ✅；推论 2 加 SDEdit 作为首个直接编辑实证；待调研方向补 SDEdit），不升 working thesis 版本号。

## Open questions / 待追

> 用户 annotation 无 🔵/🟣 显式标注（颜色全黄 / 灰）。以下由 my-summary + annotation + overview 接口推断，按 thesis 距离排序。

- [ ] **P0**：SDEdit 的"分布重合"前提（Fig 1）能否量化？给定 guide 与目标，最优 $t_0^*$ 是否可由两者加噪分布的 KL / Wasserstein 重合度预测？这是把"在哪个 $t$ 介入"从经验调参变成可计算量的入口——直接服务推论 2。
- [ ] **P0**：$t_0$（noising strength）与 [[wiki/concepts/classifier-free-guidance|CFG]] 的 $w$（guidance strength）两个旋钮在 fidelity↔controllability 平面上是否正交？两者组合（SDEdit + CFG）的行为是否可分解？
- [ ] **P1**：SDEdit 在 latent diffusion（SD img2img）上 vs pixel diffusion 上，realism↔faithfulness 曲线是否因 [[wiki/concepts/perceptual-compression|autoencoder 重建上限]]而平移？这把 LDM ingest 的"正交 fidelity 上界"与 SDEdit 的 $t_0$ 旋钮接起来。
- [ ] **P1**：noising-based（SDEdit）vs inversion-based（DDIM-inversion / Null-text）在同一编辑任务上的忠实度 / 成本帕累托前沿——thesis sideband 光谱的实验化。
- [ ] **P2**：SDEdit 用 SDE（随机）vs 用 [[wiki/concepts/probability-flow-ode|PF-ODE]]（确定性）reverse 的差异——随机性对 realism 的贡献有多大？（论文用 SDE，但 PF-ODE 版即"确定性 SDEdit"，与 DDIM img2img 更接近。）
