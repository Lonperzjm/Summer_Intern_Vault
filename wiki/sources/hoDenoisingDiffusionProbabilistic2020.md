---
type: source
title: "Denoising Diffusion Probabilistic Models (DDPM)"
aliases: [DDPM, "Ho et al. 2020", hoDenoisingDiffusionProbabilistic2020]
tags: [diffusion, generative-model, foundational]
status: stable
created: 2026-05-10
updated: 2026-05-27
raw: "[[raw/literature-notes/hoDenoisingDiffusionProbabilistic2020]]"
authors: [Jonathan Ho, Ajay Jain, Pieter Abbeel]
venue: NeurIPS
year: 2020
arxiv: "2006.11239"
---

# Denoising Diffusion Probabilistic Models (DDPM)

> Ho, Jain, Abbeel · UC Berkeley · NeurIPS 2020 · [arXiv 2006.11239](https://arxiv.org/abs/2006.11239)
> 原始文献笔记：[[raw/literature-notes/hoDenoisingDiffusionProbabilistic2020]]

## Motivation

2015 年 Sohl-Dickstein 提出的 diffusion probabilistic model 在原理上优雅，但样本质量长期落后于 GAN/autoregressive。Ho et al. 想证明：**只要训练目标与参数化方式选对，diffusion 就能产出 SOTA 图像**，并把它和 score matching with Langevin dynamics（Song & Ermon, 2019）建立形式上的等价。

## Method

### Forward process（固定的高斯加噪链）

$$
q(x_t \mid x_{t-1}) = \mathcal{N}(x_t;\, \sqrt{1-\beta_t}\, x_{t-1},\, \beta_t I)
$$

闭式一步采样性质（关键）：令 $\alpha_t := 1-\beta_t,\ \bar\alpha_t := \prod_{s\le t}\alpha_s$，则

$$
q(x_t \mid x_0) = \mathcal{N}(x_t;\, \sqrt{\bar\alpha_t}\, x_0,\, (1-\bar\alpha_t) I)
$$

可写为 $x_t = \sqrt{\bar\alpha_t}\, x_0 + \sqrt{1-\bar\alpha_t}\, \varepsilon,\ \varepsilon\sim\mathcal{N}(0,I)$。详见 [[wiki/concepts/diffusion-process]]、[[wiki/concepts/reparameterization-trick]]。

### Reverse process

学习一族高斯转移 $p_\theta(x_{t-1}\mid x_t)=\mathcal{N}(x_{t-1};\mu_\theta(x_t,t),\Sigma_\theta(x_t,t))$。当 $\beta_t$ 足够小时，反向真后验近似仍是高斯——这一点是用 Sohl-Dickstein 2015 的引理来正当化的（注：仅在 $\beta_t\to 0$ 时严格，**SDE 视角下**这是个被反复推敲的近似，见下方 Open questions）。

### 训练目标：从 ELBO 到 L_simple

变分上界（[[wiki/concepts/variational-bound-elbo]]）：

$$
\mathbb{E}[-\log p_\theta(x_0)] \le \mathbb{E}_q\!\left[-\log p(x_T) - \sum_{t\ge 1}\log\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}\right]
$$

利用 $q(x_{t-1}\mid x_t, x_0) = \mathcal{N}(x_{t-1};\tilde\mu_t(x_t,x_0),\tilde\beta_t I)$，其中

$$
\tilde\mu_t(x_t,x_0) = \frac{\sqrt{\bar\alpha_{t-1}}\beta_t}{1-\bar\alpha_t}x_0 + \frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t,\qquad \tilde\beta_t = \frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}\beta_t
$$

把每个 $L_t$ 化成 $\tilde\mu_t$ 与 $\mu_\theta$ 之间的 L2 距离。

**关键参数化（本文核心贡献）**：把 $\mu_\theta$ 重写为

$$
\mu_\theta(x_t,t) = \frac{1}{\sqrt{\alpha_t}}\!\left(x_t - \frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\,\varepsilon_\theta(x_t,t)\right)
$$

即让网络去**预测注入的噪声 $\varepsilon$**，而非直接预测均值或 $x_0$。详见 [[wiki/concepts/epsilon-parameterization]]。

进一步丢掉每个 $t$ 前的权重，得到本文实际使用的简化损失：

$$
L_\mathrm{simple}(\theta) = \mathbb{E}_{t,x_0,\varepsilon}\!\left[\big\| \varepsilon - \varepsilon_\theta(\sqrt{\bar\alpha_t}\,x_0 + \sqrt{1-\bar\alpha_t}\,\varepsilon,\ t) \big\|^2\right]
$$

这等价于带权重的 denoising score matching，从而和 [[wiki/concepts/score-matching]] / [[wiki/concepts/langevin-dynamics]] 框架联通。

### 算法（来自笔记中嵌入图）

![[hoDenoisingDiffusionProbabilistic2020-1778421475909.webp]]

- **Training**：随机抽 $t\sim\mathcal{U}\{1,T\}$、$\varepsilon\sim\mathcal{N}(0,I)$，最小化 $\|\varepsilon-\varepsilon_\theta(\cdot)\|^2$
- **Sampling**：$x_T\sim\mathcal{N}(0,I)$；对 $t=T,\dots,1$，$x_{t-1}=\frac{1}{\sqrt{\alpha_t}}(x_t-\frac{\beta_t}{\sqrt{1-\bar\alpha_t}}\varepsilon_\theta(x_t,t))+\sigma_t z$（$z\sim\mathcal{N}(0,I)$ 当 $t>1$）

### 实现要点

- 网络：基于 PixelCNN++/Transformer 的 U-Net，时间步 $t$ 经 sinusoidal embedding 注入
- $T=1000$，$\beta$ 线性从 $10^{-4}$ 到 $0.02$
- $\Sigma_\theta$ 取固定方差 $\sigma_t^2=\beta_t$ 或 $\tilde\beta_t$，效果相近

## Results

- **CIFAR-10**（[[wiki/benchmarks/cifar10]]）：Inception Score 9.46，FID **3.17**（彼时 SOTA）
- **LSUN 256×256**（[[wiki/benchmarks/lsun]]，bedroom/church/cat）：样本质量与 ProgressiveGAN 相当
- **Progressive lossy decompression**：把采样链解读为 rate–distortion 上的渐进解码，与自回归解码同构

## 关系

- 实体：[[wiki/entities/jonathan-ho]]、[[wiki/entities/pieter-abbeel]]、[[wiki/entities/uc-berkeley]]
- 方法：[[wiki/methods/ddpm]]（本论文 = 该方法的奠基）
- 概念：
  - [[wiki/concepts/diffusion-process]] —— forward + reverse 双过程定义
  - [[wiki/concepts/variational-bound-elbo]] —— ELBO 在 diffusion 中的具体形式
  - [[wiki/concepts/epsilon-parameterization]] —— 本文核心创新
  - [[wiki/concepts/score-matching]]、[[wiki/concepts/langevin-dynamics]] —— 与 NCSN 框架的等价
  - [[wiki/concepts/reparameterization-trick]] —— forward 闭式采样
- 基准：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/lsun]]
- 上游：[[wiki/methods/diffusion-2015|Sohl-Dickstein et al. 2015]]（diffusion 原型）；[[wiki/methods/ncsn|Song & Ermon 2019]]（NCSN, score-based）
- 下游：[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] (Song et al., ICLR 2021)、IDDPM (Nichol & Dhariwal 2021)、[[wiki/concepts/score-sde|Score SDE]] (Song et al. 2021)、[[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance]] (Ho & Salimans 2022)、[[wiki/methods/ldm|Stable Diffusion / LDM]] (Rombach et al. 2022) 等几乎所有现代 diffusion 工作

## 对我的 thesis 的启示

- DDPM 把"**全局生成 + 渐进精化**"这条思路第一次工程化到 SOTA 水平，后续 text-guided editing 几乎全部建立在 ε-prediction 的训练形式上。在 [[wiki/overview]] 的可变性光谱里，ε-pred 是"可演化但非主战场"的一档——thesis 应承认它是默认共识，但研究杠杆在 inversion / guidance / 条件注入 / 介入时间步。
- 对 [[wiki/overview]] 的 working thesis 已写入第一版（diffusion = 全局 + 渐进；ε-pred 是事实标准；编辑层差异主要在 inversion / guidance / 条件注入而非底层目标），可继续演化。

## Open questions / 待追

- [ ] $\beta_t$ 小 ⇒ 反向高斯近似的严格 SDE 推导（你 🔴 的疑虑）—— 应在 [[wiki/concepts/diffusion-process]] 中补"连续时间极限"小节，参考 [[wiki/concepts/score-sde]] (Song et al. 2021)
- [ ] $L_\mathrm{simple}$ 为什么经验上比加权 ELBO 更好？理论解释 / 是否与 noise schedule 选择耦合
- [ ] [[wiki/methods/diffusion-2015|Sohl-Dickstein 2015]] 的精确推导链（你 🔵 标注 §2 Background 的"待追"）
- [ ] $\Sigma_\theta$ 学习与不学习的 trade-off（IDDPM 在这条线上做了正面攻击）
