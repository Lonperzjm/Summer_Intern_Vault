---
type: source
title: "Diffusion Models Beat GANs on Image Synthesis"
aliases: [ADM, Guided Diffusion, "Dhariwal & Nichol 2021"]
tags: [diffusion, guidance, conditioning, architecture, ImageNet]
status: active
created: 2026-08-11
updated: 2026-08-11
raw: "[[raw/literature-notes/dhariwalDiffusionModelsBeat2021]]"
authors: "Prafulla Dhariwal, Alex Nichol"
venue: NeurIPS 2021
year: 2021
arxiv: "2105.05233"
---

# Diffusion Models Beat GANs on Image Synthesis

> Dhariwal & Nichol, 2021. 通常称 **ADM**（Ablated Diffusion Model）或 **Guided Diffusion**。
> Diffusion 模型首次在 ImageNet 类条件生成上全面超过 GAN（FID + recall），并提出 **classifier guidance** 作为 fidelity–diversity 旋钮。

## Motivation

2021 年以前，DDPM 已能生成不错的图像，但在 ImageNet 这种大规模数据集上 FID 仍不如 BigGAN-deep。作者认为差距来自两方面：

1. GAN 的网络架构经过多年打磨，diffusion 的 U-Net 相对朴素；
2. GAN 有 truncation trick 主动牺牲多样性换保真度，diffusion 缺少等价旋钮。

## Method

### 1. 架构改进 → ADM

在 [[wiki/methods/ddpm|DDPM]] 的 U-Net 基础上做系统化 ablation：

| 改动 | 说明 |
|---|---|
| 多分辨率 attention | 在 $32{\times}32$、$16{\times}16$、$8{\times}8$ 同时加 attention |
| Multi-head attention | ~64 channels/head |
| BigGAN-style residual up/down | 用 residual block 做分辨率变换 |
| **Adaptive Group Normalization (AdaGN)** | $\mathrm{AdaGN}(h,y)=y_s\cdot\mathrm{GN}(h)+y_b$，FiLM-style 注入 timestep + class |
| 加宽 > 加深 | 固定 FLOPs 下宽网络更划算 |

改进后的模型称为 **ADM**。训练目标不变，仍是 [[wiki/concepts/epsilon-parameterization|ε-prediction]] 的 $L_\mathrm{simple}$。

### 2. Classifier Guidance

核心思想：用外部训练的 **noise-robust 分类器** $p_\phi(y\mid x_t,t)$ 的梯度，在采样时把无条件 score 推向目标类别。

**贝叶斯分解**（与 [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] 同源）：
$$\nabla_{x_t}\log p(x_t\mid y) = \nabla_{x_t}\log p(x_t) + \nabla_{x_t}\log p_\phi(y\mid x_t).$$

**在 ε 参数化下**：
$$\hat\varepsilon(x_t,t) = \varepsilon_\theta(x_t,t) - \sqrt{1-\bar\alpha_t}\,\nabla_{x_t}\log p_\phi(y\mid x_t,t).$$

**等价的 Gaussian 均值平移视角**（论文 Section 4 的漂亮推导）：

对 reverse transition $p_\theta(x_t\mid x_{t+1})=\mathcal N(\mu,\Sigma)$ 的后验做一阶 Taylor 展开：
$$p(x_t\mid x_{t+1},y)\approx\mathcal N\!\big(\mu+s\,\Sigma\,\nabla_{x_t}\log p_\phi(y\mid x_t),\;\Sigma\big).$$

### 3. Guidance Scale $s$

$$s\,\nabla_x\log p(y\mid x) = \nabla_x\log p(y\mid x)^s$$

等价于从 $p(x)\,p(y\mid x)^s$ 采样。$s>1$ sharpen 后验：

$$s\uparrow\;\Rightarrow\;\text{fidelity/precision}\uparrow,\quad\text{diversity/recall}\downarrow.$$

这是 diffusion 版的 **fidelity–diversity tradeoff knob**，与 GAN 的 truncation trick 功能对等。

### 4. 噪声鲁棒分类器

分类器不是在 clean image 上训练——它在 diffusion 的每个 noise level $t$ 下都接受 $(x_t, t)$ 并预测 $y$。架构复用 ADM 的 downsampling trunk + attention pool。

## Results

ImageNet class-conditional generation（ADM-G = ADM + classifier guidance）：

| 分辨率 | FID↓ | Recall↑ | 对比 BigGAN-deep |
|---|---|---|---|
| 128×128 | **2.97** | — | — |
| 256×256 | **4.59** | **0.52** | 6.95 / 0.28 |
| 512×512 | **7.72** | — | — |

加 upsampling diffusion 后：256×256 FID **3.94**，512×512 FID **3.85**。

同时以 25 步 DDIM 即可 match BigGAN-deep，说明 classifier guidance 与加速采样兼容。

## 关系

### 直接相关

- [[wiki/concepts/classifier-guidance]]：本文是该概念的**原文**，给出了完整工程化
- [[wiki/concepts/classifier-free-guidance]]：直接后续（Ho & Salimans 2022），把分类器引导项内化进同一网络
- [[wiki/concepts/score-matching]] / [[wiki/concepts/score-sde]]：classifier guidance 的数学框架（$\nabla\log p(x|y)=\nabla\log p(x)+\nabla\log p(y|x)$）
- [[wiki/concepts/epsilon-parameterization]]：训练目标不变，guidance 只在采样时修改 $\hat\varepsilon$
- [[wiki/concepts/energy-guidance]]：把 $\log p_\phi(y|x)$ 推广为任意可微能量（[[wiki/methods/egsde|EGSDE]]、[[wiki/methods/dps|DPS]]）
- [[wiki/concepts/training-free-guidance]]：[[wiki/methods/freedom|FreeDoM]] 把 classifier guidance 改为 $\hat x_0$ 点估计 + 现成判别器，免去噪声鲁棒训练

### 实体

- [[wiki/entities/jonathan-ho]]：DDPM 作者，后续提出 CFG
- OpenAI：本文作者机构，代码发布为 `openai/guided-diffusion`

### 方法

- [[wiki/methods/ddpm]]：本文的 baseline 与训练框架
- [[wiki/methods/ddim]]：论文 Section 4 也给出了 DDIM 下的 guided sampling 形式

## 对我的 thesis 的启示

1. **不改变 working thesis**。本文是 overview 推论 1（可变性光谱）与推论 2（fidelity↔controllability 旋钮）的**历史起点**，但其内容已通过 Score SDE ingest 被概念页吸收。此次 ingest 的意义是**给 classifier-guidance 概念页补上原文实证与架构细节**。

2. **对 thesis 实验方向的间接价值**：classifier guidance 的 "pretrained prior + external gradient" 模板正是 [[wiki/concepts/energy-guidance|energy guidance]] / [[wiki/methods/freedom|FreeDoM]] 的直系祖先。当前实验主线（reject-and-skip）不直接用 classifier guidance，但未来若回到 energy-guidance 路线，本文的 "guidance scale 与 FID/Recall 的 trade-off" 实验设计是直接的参考模板。

3. **AdaGN 作为 conditioning 注入机制**：FiLM-style 的 AdaGN 是后续所有 DiT / SD3 / FLUX 中 adaptive layer norm 的直接前身——如果 thesis 需要设计新的 sideband conditioning 注入点，AdaGN → AdaLN 这条线值得回顾。

## 待后续 ingest

- [x] Ho & Salimans 2022（CFG 原文）—— ✅ 已 ingest [[wiki/sources/hoClassifierFreeDiffusionGuidance2022]]，与本文形成完整 guidance 脉络
- [ ] Nichol & Dhariwal 2021（IDDPM）——学习 $\Sigma_\theta$、改进 NLL
