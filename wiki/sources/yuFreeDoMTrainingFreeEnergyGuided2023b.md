---
type: source
title: "FreeDoM: Training-Free Energy-Guided Conditional Diffusion Model"
aliases: [FreeDoM, "Yu et al. 2023", training-free energy-guided diffusion]
tags: [diffusion, guidance, energy-guidance, training-free, clean-estimate, editing]
status: stable
created: 2026-06-23
updated: 2026-06-23
raw: "[[raw/literature-notes/yuFreeDoMTrainingFreeEnergyGuided2023b]]"
authors: [Jiwen Yu, Yinhuai Wang, Chen Zhao, Bernard Ghanem, Jian Zhang]
venue: ICCV 2023
year: 2023
arxiv: "2303.09833"
---

# FreeDoM: Training-Free Energy-Guided Conditional Diffusion Model

> 文献笔记：[[raw/literature-notes/yuFreeDoMTrainingFreeEnergyGuided2023b]] · arXiv [2303.09833](http://arxiv.org/abs/2303.09833) · Yu et al. · ICCV 2023 · PKU + KAUST
> 这是 [[wiki/concepts/training-free-guidance|training-free guidance]] 的代表作，也是你 energy-guidance 候选最近的 prior art —— **clean-estimate 路线、和 [[wiki/methods/egsde|EGSDE]] 的 noisy-aligned 正好相反**。

## 一句话

不训练任何新东西，用**现成的、时间无关的、作用于干净图的**模型（人脸检测 / CLIP / 分割 / ArcFace / Gram 风格 / 低通）构造能量；通过 Tweedie 把 $x_t$ 估成 $\hat x_0$，把能量评在 $\hat x_0$ 上，梯度 $\nabla_{x_t}E$ 引导采样。

## Motivation

🟡 p.23175：现有条件扩散一旦要换新条件就得**重训/微调**，贵且不便（"Once a new target condition is needed, they have to retrain or finetune"）。FreeDoM 要的是：**任意现成 clean-image 模型即插即用、零训练**地做条件生成。难点（🟡 p.23177）：**带噪图上几乎找不到现成预训练网络**——所以不能直接对 $x_t$ 打分。

## Method

### 1. 能量引导的起点（🟡 p.23176 eq4）
$$p(c\mid x_t)=\frac{\exp\{-\lambda E(c,x_t)\}}{Z},\qquad \nabla_{x_t}\log p(c\mid x_t)\propto-\nabla_{x_t}E(c,x_t).$$
> 🔴 用户批注（eq4）："**不够严格，$Z$ 没算**"。一针见血——这里的 $\propto$ 把 $\lambda$ 和归一化 $Z$ 都藏了。严格处理见 [[wiki/concepts/conditional-diffusion]] §2：用能量**重加权先验**（$Z$ 对 $x$ 积分、$x$-梯度为 0）才干净，FreeDoM 这步是工程化的略写。

### 2. 关键近似：clean-estimate（🟡 p.23177 eq7/eq9）
带噪能量没法直接算，于是 marginalize 再用点估计：
$$D_\phi(c,x_t,t)\approx\mathbb E_{p(x_0\mid x_t)}[D_\theta(c,x_0)]\ \xrightarrow{\text{点估计}}\ E(c,x_t)\approx D_\theta(c,\hat x_{0\mid t}).$$
其中 Tweedie 估计（用户 my-summary #1）：
$$\hat x_{0\mid t}=\frac{1}{\sqrt{\bar\alpha_t}}\big(x_t+(1-\bar\alpha_t)s(x_t,t)\big).$$
**这就是 clean-estimate-level energy guidance**——和 DPS 同构（$x_t\to\hat x_0\to E(\hat x_0;c)\to\nabla_{x_t}E$），但把"观测一致性"推广成任意条件能量。

### 3. 能量 = 距离族（🟡 p.23178 eq11 / eq12）
$$E(c,x_t)\approx \mathrm{Dist}\big(P_{\theta_1}(c),P_{\theta_2}(\hat x_{0\mid t})\big),$$
现成模型给距离：CLIP 文本距离 / 分割距离 / 草图距离 / ArcFace 身份距离 / Gram 风格距离 / 低通结构距离。多条件加权（eq12）$E=\sum_i\eta_i D_{\theta_i}$，**假设条件独立**。

### 4. Time-travel 策略（🟡 p.23178）
只在**中间"semantic stage"**用 time-travel（回跳重采）：早期 chaotic 阶段 $\hat x_{0\mid t}$ 太糊、引导没意义；后期 refinement 阶段变化太小、没必要。

### 5. 采样式与 latent 版（my-summary #4）
$$x_{t-1}=m_t-\rho_t\,\nabla_{x_t}E(\hat x_{0\mid t}(x_t),c),\qquad \text{latent: } z_t\to z_{0\mid t}\to\hat x_0=\mathrm{Dec}(z_{0\mid t})\to E\to\nabla_{z_t}E.$$
因此能**外挂到 SD / ControlNet** 上，形成"训练式内部条件接口 + 免训练外部能量接口"的混合控制。

## Results

- 单一框架覆盖多条件：face ID（ArcFace）、style（Gram）、text（CLIP）、segmentation、sketch、landmark 等，**全部零训练**。
- 可叠加在 Stable Diffusion / ControlNet 上做组合控制。
- **局限**（用户 my-summary #5）：① 省训练但**增推理成本**（每步能量反传 + time-travel 加步）；② ImageNet 等大域上**细粒度结构控制弱**，Canny 边缘即使 time-travel 也可能 poor guidance；③ 多条件**独立假设**，条件冲突时结果差。

## 关系

- 概念归属：[[wiki/concepts/training-free-guidance|training-free guidance]]（代表作）、[[wiki/concepts/energy-guidance]]、[[wiki/concepts/conditional-diffusion]]（§3–§4 的具体化）；推广自 [[wiki/concepts/classifier-guidance]]。
- 方法页：[[wiki/methods/freedom]]。
- **最近邻 DPS**：结构同构（clean-estimate + 反传）；FreeDoM = DPS 从"反问题观测一致性"推广到"任意现成条件能量"。
- **与 [[wiki/methods/egsde|EGSDE]] 正相反**：EGSDE 是 noisy-aligned（噪化源 + 重训 $E_s$ + MC 期望）；FreeDoM 是 clean-estimate（$\hat x_0$ + 现成模型 + 点估计）。这对照是 [[wiki/concepts/conditional-diffusion]] §5 那张表的两行。
- 被 TFG 收编：是 [[wiki/concepts/training-free-guidance]] 设计空间里"点估计 + time-travel"的特例。
- 可外挂的底座：[[wiki/methods/ldm|LDM]] / [[wiki/methods/controlnet|ControlNet]]。
- 作者 / 机构：Jiwen Yu、Yinhuai Wang、Chen Zhao、Bernard Ghanem（KAUST）、Jian Zhang（PKU）。

## 对我的 thesis 的启示

> 🟣 这篇是你 energy-guidance 候选（[[research/ideas]]）**最近的 prior art**——你想做的"clean-estimate + 现成判别器"，FreeDoM 已经在 diffusion 上做了，且被 TFG 收编。

- **占位确认**：clean-estimate-level energy guidance 在 diffusion 上 = FreeDoM/DPS 已占。你的差异化只能往 **flow 角度**（RF 的 $\hat x_0=x_t-t v_\theta$ 偏差更小）或 FreeDoM 三个明确局限（推理成本 / 细结构 / 多条件冲突）里找窄 niche。
- **🔴 批注的价值**："eq4 的 $Z$ 没算"——FreeDoM 自己的理论略写，是可被你 [[wiki/concepts/conditional-diffusion]] 的严格 $Z'$ 处理补的一个小口子（但更像写作严谨性，不一定是发文点）。
- 不动 [[wiki/overview]] working thesis 版本号；这篇巩固"energy-guidance 红海"判断，sliver 仍待师兄定。

## 我的 takeaways

1. FreeDoM = 现成 time-independent 模型 + Tweedie $\hat x_0$ + 点估计能量 → 免训练条件生成。
2. 明确 clean-estimate 路线，结构同 DPS，和 EGSDE 的 noisy-aligned 相反。
3. 能量 = 距离族（CLIP/ArcFace/分割/草图/Gram/低通），多条件简单加权。
4. time-travel 只在 semantic 中段用；可外挂 SD/ControlNet。
5. 局限（推理成本 / 细结构弱 / 多条件冲突）= 后续 training-free 引导的改进方向。

## Open questions / 待追（🔵）

- [ ] DPS 原文 ingest（FreeDoM 的最近邻，结构同构）
- [ ] TFG 原文 ingest（把 FreeDoM/DPS/LGD/UGD/MPGD 收进一个设计空间）
- [ ] FreeDoM 的"细结构控制弱"在 RF/FM 上是否缓解（接 flow 假设）
- [ ] 多条件冲突的非独立建模（FreeDoM eq12 的独立假设的改进口）
