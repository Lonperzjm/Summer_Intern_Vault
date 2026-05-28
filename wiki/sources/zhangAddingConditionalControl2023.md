---
type: source
title: "Adding Conditional Control to Text-to-Image Diffusion Models"
aliases: [ControlNet, ControlNet 原文, "Zhang et al. 2023", "Zhang, Rao & Agrawala 2023"]
tags: [diffusion, control-injection, sideband, latent-diffusion, text-to-image, editing-related, foundational]
status: stable
created: 2026-05-28
updated: 2026-05-28
raw: "[[raw/literature-notes/zhangAddingConditionalControl2023]]"
authors: [Lvmin Zhang, Anyi Rao, Maneesh Agrawala]
venue: ICCV 2023
year: 2023
arxiv: "2302.05543"
---

# Adding Conditional Control to Text-to-Image Diffusion Models

> 文献笔记：[[raw/literature-notes/zhangAddingConditionalControl2023]] · arXiv [2302.05543](http://arxiv.org/abs/2302.05543) · Zhang, Rao & Agrawala 2023 (ICCV)
> 本页是 **control/sideband-injection** 派系的奠基论文，是 [[wiki/overview]] 主要派系新增第 5 类的首篇代表作。具体附着于 [[wiki/entities/stable-diffusion|Stable Diffusion]] v1.5 之上。

## 一句话

**不改 LDM 的训练目标、不动 SD backbone**，给 [[wiki/entities/stable-diffusion|Stable Diffusion]] 的 U-Net **encoder + middle** 段做一份 **trainable copy**，副本输入 = noisy latent $z_t$ ⊕ 条件图编码 $c_f$，副本各层输出**经 [[wiki/concepts/zero-convolution|zero convolution]]** 加回原 U-Net 的 12 条 skip + 1 个 middle——这是把空间对齐的强条件（edge / depth / pose / segmentation / scribble …）注入预训练 T2I 模型的事实标准。"sideband 注入"这条研究通道与 [[wiki/concepts/cross-attention|cross-attention 注入]] 完全正交、互不替代。

## Motivation

LDM/SD 的 cross-attention 是 **token 类条件**的通用接口（文本、layout、class），但对**空间对齐**强条件（depth map、Canny edge、人体 pose 骨架、分割图、scribble），三类前置路线都不够好：

1. **从头训一个 conditional T2I**：billion 级数据 + GPU-month 成本，学界做不动。
2. **直接 fine-tune SD 加新条件**：用 < 50k 条件数据 fine-tune 容易**过拟合 + 灾难性遗忘**（🟡 p.2）——你想加 Canny 控制，结果整个文本理解能力崩了。
3. **Adapter / LoRA 加在文本/cross-attn 内**：与 cross-attn 接口绑定，对**空间对齐**的强条件不自然（条件是 spatial grid 不是 token 序列）。

作者要的：(i) **冻结 SD 主干**（保住所有预训练能力）；(ii) 在 backbone 之**侧**加 trainable module 学条件→去噪场的多尺度残差；(iii) 初始化时不破坏 SD 的输出。

🟡 p.1：「locks the production-ready large diffusion model」+「zero convolutions」是论文的两条主张句。🟡 p.2 (1)：「propose ControlNet, a neural network architecture that can add spatially localized input conditions to a pretrained text-to-image diffusion model via efficient finetuning」。

## Method

### 0. 总体管线

**原文 Fig 3（SD + ControlNet 全景集成）**：

![[zhangAddingConditionalControl2023-1779969462771.webp]]

**逐项对照 Fig 3**：

- **左栏（SD，🔒 冻结）**：Text/Time encoder + 4 个 SD Encoder Block（A 64² / B 32² / C 16² / D 8²，**各 ×3** = 12 个 block）+ SD Middle Block 8² + 对称的 4 个 SD Decoder Block（D / C / B / A，**各 ×3**）。输入 $z_t$、输出 $\varepsilon_\theta(z_t, t, c_t, c_f)$。
- **右栏（ControlNet，trainable）**：条件 $c_f$ 进入前先过一个 zero conv，与 $z_t$ 相加后进入 trainable copy；trainable copy 复制 SD 的 **4 个 encoder block 各 ×3 = 12 个 + 1 个 middle block**（**不复制 decoder**——这是减半参数、且只学"如何向 skip 注入残差"的关键，图中右栏到 Middle 就截止）。
- **注入（右→左的 13 条蓝色箭头）**：副本 4 个分辨率层级各 ×3 = 12 路 encoder block 输出 + 1 路 middle 输出，分别经一层 **[[wiki/concepts/zero-convolution|zero conv]]** 后**加性**注入到 SD decoder 的对应 12 条 skip 与 middle 上。
- **训练动作**：只更新右栏（trainable copy + 7 处 zero conv + 条件 stem）；左栏完全冻结。

> 简言之：图里**🔒 的全部冻结，蓝色虚线框内的全部训练**；右栏只到 middle 截止、decoder 没有副本——这两条是最容易看漏的细节。

**Fig 3 没显式画出的工程细节**：

- 条件图 $c_f$ 是 $512^2\times 3$ 的图像（Canny / depth / pose 等），进入右上角 zero conv 之前还要经过一个**小卷积栈**：4 层 $4\times 4$ conv、stride 2、channels 16→32→64→128，把 $512^2$ 下采样到 $64^2$ 以匹配 SD latent 的空间分辨率（图中没单独画这层，融在 "Condition $c_f$" 节点里）。
- "trainable copy" 在初始时 **完全复制** SD encoder + middle 的预训练权重（非随机初始化）——这是 ControlNet 能用 50k 数据收敛的另一关键：副本不是从零学，是从一个"已经会去噪的"起点出发学条件残差。

### 1. 训练目标（🟡 p.4 公式）

$$
\mathcal L = \mathbb E_{z_0, t, c_t, c_f, \varepsilon\sim\mathcal N(0,I)}
\big[\|\varepsilon - \varepsilon_\theta(z_t, t, c_t, c_f)\|^2\big].
$$

- $c_t$：文本条件（走 SD 原有 cross-attention，**不动**）；
- $c_f$：新加入的空间对齐条件（走 sideband）；
- $\varepsilon_\theta$：组合网络（SD frozen + ControlNet trainable copy）。

与 [[wiki/methods/ddpm|DDPM]]/[[wiki/methods/ldm|LDM]] 的 ε-pred MSE **同构**——这正是用户 my-summary 第 2 条强调的"训练目标不变"。ControlNet 只在 $\varepsilon_\theta$ 这个**函数**上加了挂件，没动**损失泛函**的形式。

### 2. Zero Convolution：稳定化关键（🟡 p.2/p.3）

**原文 Fig 2（ControlNet 抽象 block，把任意 frozen 神经网络块 $x\to y$ 升级为 $x\to y_c=y+\text{ZeroConv}(\text{TrainableCopy}(x+\text{ZeroConv}(c)))$）**：

![[zhangAddingConditionalControl2023-1779969426205.webp]]

左 (a) Before：原 frozen block $y=\text{block}(x)$；右 (b) After：frozen block 不动，旁挂一个 trainable copy，输入是 $x+\text{ZeroConv}(c)$，输出再过 ZeroConv 加回 $y$。初始化时两个 ZeroConv 均输出 0，所以 $y_c=y$（恒等）；训练后 ZeroConv 学到非零权重，控制 residual 才生效。这是 Fig 3 中右栏每条"zero conv"箭头的**抽象单元**。

**定义**：$1\times1$ 卷积，**weights = 0、bias = 0** 初始化（但 weight 是 trainable 参数）。详见 [[wiki/concepts/zero-convolution]]。

**为什么不破坏 SD**：训练第 0 步，副本所有 zero-conv 输出 = 0 → sideband 残差 = 0 → 整网输出 = SD 原输出，损失 = SD 在该 batch 上的损失（不被随机副本扰动）。

**为什么梯度不死**：zero-conv 的输入 $\ne 0$（来自副本前一层的非零激活），所以 $\partial\mathcal L/\partial W = X^\top G \ne 0$，第一步梯度更新即可让 $W$ 偏离 0。原文 §3.1 给出推导（一旦 $W$ 第一步偏离 0，后续梯度立刻有 $\frac{\partial Y}{\partial X}$ 项流回上游副本）。

这是 LoRA / 普通 adapter 的"**初始化使新模块为恒等映射**"思想的极端版——LoRA 用 $A=0,B=\text{Gaussian}$ 让 $\Delta W=BA=0$，ControlNet 直接归零 zero-conv 整层。

### 3. Sudden Convergence Phenomenon（🔴 p.5）

作者观察到：模型**不渐进**学到控制，而是在 < 10k 步内**突然**从"忽略 $c_f$"切换到"精确遵循 $c_f$"。论文只描述、无解释（Fig 4）。

> ⚠️ 用户在 my-summary 与 annotation 中将其标 🔴（异议 / 可疑）。一种猜想（与 ControlNet 的初始化 + 梯度结构一致）：早期 zero-conv 把残差压成近零，损失主要由 SD 主路（已收敛）降低；当 zero-conv 累积到一个"足够大"的尺度，控制残差才开始对损失有梯度贡献，触发**相位转换**。这是 thesis 可立的训练动力学诊断变量（详见 open questions）。

### 4. 工程实现要点

- **条件图编码 stem**：4 层 conv（$4{\times}4$ kernel, stride 2, channels 16/32/64/128），把 $512^2\times 3$ 条件图打到 $64^2\times 320$ 的 latent grid 上，与 $z_t$ 同形相加。
- **训练**：单 RTX 3090 / A100 即可；50k–1M 数据；prompt dropout（50% 替换为空 prompt）让模型学到 unconditional 控制能力，配合 SD 的 [[wiki/concepts/classifier-free-guidance|CFG]]。
- **采样**：与原 SD 完全一致（DDIM / Euler / PLMS / etc. 皆可），SD 与 ControlNet 联合前向。

## Results

- **支持的空间条件**（§4 / §5）：Canny edge、Hough lines、HED soft edges、user scribbles、人体 pose（OpenPose）、ADE20K segmentation、depth maps、normal maps、cartoon line drawings、Anime line drawings 等 **8+ 类**——同一架构、不同条件图，分别训 ControlNet 权重。
- **数据规模**：用户 prompt 中标的"50k 也能稳"是 [[wiki/sources/zhangAddingConditionalControl2023]] 的关键经验结论——多个条件用 50k–200k 数据即可拿到高质量结果（远小于 SD 本身 LAION 训练规模）。
- **Single / multi-condition**（🟡 p.1）：可同时挂载多个 ControlNet（如 pose + depth），加性 residual 直接叠加。
- **Prompt 可选**：even **without prompt**（$c_t$=空）也能从条件图生成合理内容——证明 sideband 学到的"条件→空间分布"知识独立于文本路径。
- **Ablation（🟡 p.6 §4.2）**：用户批注精确——(1) 浅层小 adapter 不够，复杂空间条件需要**深层 trainable copy**（即副本必须有 SD encoder 的全部 12 层而非浅几层）；(2) zero conv **不能替换** 为普通 $1\times 1$ conv（高斯初始化训练崩溃）。

## 关系（与已有 wiki 的关联）

- **直接前置**：[[wiki/entities/stable-diffusion]]（ControlNet 在原文里附着于 SD v1.5）；[[wiki/methods/ldm|LDM]]（SD 的方法基础）；[[wiki/methods/ddpm|DDPM]] / [[wiki/methods/ddim|DDIM]]（训练目标与采样器无改）。
- **本页直接催生的新页**：[[wiki/methods/controlnet]]（方法主页）；[[wiki/concepts/zero-convolution]]、[[wiki/concepts/sideband-conditioning]]；[[wiki/entities/lvmin-zhang]]、[[wiki/entities/maneesh-agrawala]]、[[wiki/entities/anyi-rao]]、[[wiki/entities/stable-diffusion]]（具名模型）。
- **vs cross-attention 注入**（[[wiki/concepts/cross-attention]]）：**完全正交**。Cross-attention 在 U-Net 内部把 token 类条件 K/V 化注入；ControlNet 在 U-Net 外部把空间对齐条件 sideband 化注入。两者通常**共存**——文本走 cross-attn（SD 原有），depth/pose 走 ControlNet。详见 [[wiki/concepts/sideband-conditioning]]。
- **vs Concat 注入**（[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM 原文]] §3.3 末段）：LDM 对空间对齐条件原本是直接 concat 到 noisy latent 通道维；ControlNet 把这条路升级为"复制 encoder + sideband 加性注入"——更稳、可加性更强、不需要重训 SD。
- **vs [[wiki/concepts/classifier-free-guidance|CFG]]**：正交。ControlNet 仍可与 CFG 联用（按文本 prompt 走 SD 原有 CFG 路径；按空间条件走 sideband）；论文末 §6 提到也可做 conditional/unconditional ControlNet 的 CFG 式外推，但非标准用法。
- **vs LDM 推论 1 "动条件注入必然动 backbone 内部"**：ControlNet 给这条**精确变奏**——它**不修改 backbone 参数**（frozen），但**克隆了一份 backbone 结构**作为 sideband。所以"动 backbone 内部"在 ControlNet 上读作"复制 backbone 内部并外挂"，这是 thesis 可借的一个新刻画。
- **人物 / 机构**：[[wiki/entities/lvmin-zhang]]、[[wiki/entities/anyi-rao]]、[[wiki/entities/maneesh-agrawala]]；[[wiki/entities/stanford]]。
- **下游 / 并行同期工作（仍待 ingest）**：T2I-Adapter（Mou et al. 2023，更小的 adapter 思路）、IP-Adapter（Ye et al. 2023，image prompt 注入）、GLIGEN（Li et al. 2023，bounding box / layout 条件）。

## 对我的 thesis 的启示

按用户确认：将"**frozen backbone + trainable sideband 是 thesis 在资源约束下的核心可行抽象**"作为 🟣 级 thesis-implication 处理。

- **方法学层面的 thesis-relevant claim**：在 SD/SD3/FLUX 的 backbone 整段冻结（这一假设在学界资源约束下事实成立）的前提下，研究杠杆几乎全部坐落在 **sideband + 注入接口** 这一档。可把 thesis 现有的"inversion-based / attention-injection / control-injection" 三类编辑方法重新归到这一统一抽象之下：**它们都是不动 backbone 参数的 sideband 操作**，差别只在 sideband 接入的位置（注意力图 / skip connection / 初始噪声）与训练成本（zero-shot inversion / 小数据 fine-tune / 50k–200k 条件训练）。这条统一化能直接进 [[research/thesis]] 的"方法论范围"一章。
- **对 [[wiki/overview]] 可变性光谱的影响**：ControlNet 没动训练目标、没动 backbone 类型、没动采样器、没动压缩层（LDM 的 $\mathcal E,\mathcal D$ 也是冻结的）——它只在"研究杠杆 → 条件注入通道"这一档新增了一种**通道**（sideband 副本 + zero-conv 残差注入），与 cross-attention 这条通道**正交且共存**。这是推论 1 修正(iii)"动条件注入往往意味着动 backbone 内部"的**精确变奏**：ControlNet 不改 backbone 参数，但**克隆了 backbone 内部结构**作为外挂——所以这条修正可以重写为"动条件注入往往意味着**触及** backbone 内部"（而不一定是 modify in-place）。
- **对推论 2"在哪个 $t$ 注入"的扩展**：ControlNet 在**所有** $t$ 都加 residual（sideband 是时不变结构），这与 [[wiki/concepts/classifier-free-guidance|CFG]] 的"逐 $t$ 引导"形成对照。Thesis 里可立的新假设：**"全 $t$ sideband 注入"vs"逐 $t$ guidance 注入"的 fidelity↔controllability 行为差异**——若两者皆可用，且 ControlNet 经验上比 prompt-only 编辑更**结构保真**，则 trade-off 形态可能由"sideband-injection 牢、guidance 灵活"这条经验律支撑。
- **对推论 3"加速"的中性**：ControlNet 不直接关 NFE，但因 SD 主路冻结，整网联合采样的 NFE 与原 SD 同——只多约 50% 的计算（副本的 encoder+middle 前向）。在 SD3/FLUX (RF 一族) 上是否能保持同样的稳定性、是否需要 reflow-friendly 的 sideband 设计，是开放问题。
- **Sudden convergence phenomenon 作为诊断变量**：它把"训练 ControlNet"和"训练普通 fine-tune"的动力学差异暴露出来——thesis 里可作为 inversion-based vs control-injection 类编辑的训练稳定性对照实验。

> 拟据此**轻量更新** [[wiki/overview]]（推论 1 修正(iii) 措辞调整、主要派系新增第 5 类 control-injection、推论 2 加 sideband vs guidance 的 caveat 注），不升 working thesis 版本号。

## Open questions / 待追

> 用户说明：本篇 annotation 同 LDM ingest，颜色全黄（"太麻烦了"），无 🔵/🟣 显式标注。以下问题由 my-summary + annotation + overview 接口处推断而出，按 thesis 距离排序。

- [ ] **P0**：Sudden convergence phenomenon 的训练动力学解释——zero-conv 尺度 $\|W_{zc}\|$ 与控制残差 norm 的时间演化曲线是否能预测相位转换点？这是 thesis 可立的一个干净诊断实验，且与 inversion-based 编辑的稳定性诊断有共享方法论。
- [ ] **P0**：在 SD3 / FLUX (RF backbone) 上 ControlNet 的可迁移性——SD3 改 backbone 为 DiT、训练目标为 RF/FM，sideband + zero-conv 的稳定性是否依赖 SD U-Net 的 skip 结构？BFL 的 FLUX-ControlNet 是否给出可比的现象学？（同时也是 overview 主要派系 → flow-matching-based 的下一个落点）
- [ ] **P1**：ControlNet vs LoRA vs Adapter 的统一对比——三者都是 "frozen backbone + trainable sideband" 的实例，差异只在 sideband 接入位置（skip / attn 内部低秩 / FFN 内部 bottleneck）。这是 [[wiki/concepts/sideband-conditioning]] 该走的方向，但 LoRA / Adapter 原文未 ingest。
- [ ] **P1**：T2I-Adapter（Mou et al. 2023）与 ControlNet 的精确异同——T2I-Adapter 用更小的 adapter（不复制 SD encoder），但报告的控制质量略弱。这是"sideband 容量 vs 控制质量"trade-off 的另一个实验点。
- [ ] **P2**：multi-condition 叠加时 sideband 残差的可加性是否真线性？两个 ControlNet 同时挂载时是否出现冲突（如 pose 与 depth 强约束矛盾时谁赢）？论文未做详细 ablation。
- [ ] **P2**：condition image 的"对齐先验"——ControlNet 假设 $c_f$ 与目标图像**像素对齐**（Canny 来自目标本身）。当 $c_f$ 仅"结构相似"（如外部 pose 移植到不同体型），sideband 的几何敏感度有多强？这接近编辑场景，仍未在论文中触及。
