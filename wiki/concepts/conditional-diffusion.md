---
type: concept
title: Conditional Diffusion（条件扩散 / 条件 score 的拆解）
aliases: [conditional diffusion, 条件扩散, conditional score, guidance derivation]
tags: [diffusion, guidance, conditioning, score-sde]
status: active
created: 2026-06-23
updated: 2026-06-23
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]", "[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]"]
---

# Conditional Diffusion（条件扩散 / 条件 score 的拆解）

> 概念母页。所有 guidance 方法（[[wiki/concepts/classifier-guidance|classifier]] / [[wiki/concepts/classifier-free-guidance|CFG]] / [[wiki/concepts/energy-guidance|energy]] / [[wiki/methods/egsde|EGSDE]] / [[wiki/methods/freedom|FreeDoM]]）都长在这条链上：**贝叶斯拆条件 score → Tweedie 取 $\hat x_0$ → 点估计 → 能量梯度**。
> 源起用户手写推导 `raw/notes/conditional diffision.md`，此页是其 ingest 产物 + 补全。

## 1. 主定理：贝叶斯拆条件 score

要从 $p_t(x\mid c)$ 采样，反向 SDE 只需把 score 换成条件 score。对它做贝叶斯（$p_t(c)$ 与 $x$ 无关、梯度为 0）：
$$\boxed{\ \nabla_x\log p_t(x\mid c)=\underbrace{\nabla_x\log p_t(x)}_{\text{无条件 score（现成网络）}}+\underbrace{\nabla_x\log p_t(c\mid x)}_{\text{引导项}}\ }$$

## 2. 能量化两种写法（含归一化的坑）

把引导项写成能量。**有两种 framing，区别全在归一化常数 $Z$ 依赖谁。**

### 写法 A：似然式 $p(c\mid x)=\exp(-E)/Z$（classifier-guidance 路）
$$\nabla_x\log p(c\mid x)=-\nabla_x E(x,c,t)-\underbrace{\nabla_x\log Z(x,t)}_{\text{一般}\neq 0},\quad Z(x,t)=\int e^{-E}\,\mathrm dc.$$
只有当 $Z$ **不依赖 $x$**（例如强行要求 $Z\equiv 1$）时 $\nabla_x\log Z=0$ 才成立——**这是个真实假设，不是自动的**。

### 写法 B：能量重加权先验 $p(x\mid c)\propto p(x)e^{-E}$（EGSDE 路，更干净）
$$\log p_t(x\mid c)=\log p_t(x)-E(x,c,t)-\log Z_t(c),\quad Z_t(c)=\int p_t(x)e^{-E}\,\mathrm dx.$$
这里 $Z_t(c)$ 是**对 $x$ 积分**出来的，只依赖条件 $c$、不依赖当前变量 $x$，所以 $\nabla_x\log Z_t(c)=0$ **天然成立**，直接得
$$\boxed{\ \nabla_x\log p_t(x\mid c)=s(x,t)-\nabla_x E(x,c,t)\ }$$
**符号是减号**（$\log e^{-E}=-E$）。EGSDE 用的就是写法 B。

> 💡 用户洞见（note 第 3 行）：图像翻译（cat→dog）里 $p(x\mid c)$ 不见得比 $p(c\mid x)$ 简化——两个方向的似然都不平凡。所以**别纠结哪个方向**，直接用写法 B 的能量重加权先验，躲开 $Z$ 的归一化纠结。

## 3. Tweedie：从 $x_t$ 拿后验均值 $\hat x_0$

引导项里的 $p(c\mid x_t)$ 是"对带噪图打分"，往往没有。marginalize 到干净图：$p(c\mid x_t)=\mathbb E_{p(x_0\mid x_t)}[p(c\mid x_0)]$，需要 $\hat x_0$。由 $p(x_t\mid x_0)=\mathcal N(\sqrt{\bar\alpha_t}x_0,(1-\bar\alpha_t)I)$ 推：
$$\boxed{\ \hat x_0:=\mathbb E[x_0\mid x_t]=\frac{x_t+(1-\bar\alpha_t)\nabla_{x_t}\log p_t(x_t)}{\sqrt{\bar\alpha_t}}=\frac{x_t-\sqrt{1-\bar\alpha_t}\,\epsilon_\theta}{\sqrt{\bar\alpha_t}}\ }$$
预训练 $\epsilon_\theta$ 直接给出，**不用额外训练**。flow/RF 版：$\hat x_0=x_t-t\,v_\theta$。

## 4. 点估计：丢掉期望，得到能量引导

用 $\hat x_0$ 这**一个点**替掉期望（Jensen 近似）：
$$\nabla_{x_t}\log p(c\mid x_t)\approx\nabla_{x_t}\log p(c\mid\hat x_0)=-\Big(\tfrac{\partial\hat x_0}{\partial x_t}\Big)^{\!\top}\nabla_{\hat x_0}E(\hat x_0,c).$$
- $\partial\hat x_0/\partial x_t$ 含 $\partial\epsilon_\theta/\partial x_t$ → **反传过网络**（[[wiki/methods/freedom|FreeDoM]] / DPS 的主要开销）。
- Jensen gap $=p(c\mid\hat x_0)-\mathbb E[p(c\mid x_0)]$ = 点估计偏差，高噪声下变大。

## 5. 两类实例（noisy-aligned vs clean-estimate）

| | noisy-aligned | clean-estimate |
|---|---|---|
| 怎么对付带噪 $x_t$ | 把参照也噪化到同 $t$，对噪声求期望（MC） | 估 $\hat x_0$，在干净空间评现成模型 |
| 代表 | [[wiki/methods/egsde\|EGSDE]]（$E_s$ 须重训 noise-aware 分类器） | [[wiki/methods/freedom\|FreeDoM]] / DPS（diffusion）；[[wiki/methods/fmps\|FMPS]]（flow，速度↔score 桥） |
| 这一页第几节 | §2 写法 B + 噪化源做 MC | §3 Tweedie + §4 点估计 |

> EGSDE 的能量梯度 $\nabla_{y_t}\mathcal E$ 完整展开（余弦项 + 低通项）见 [[wiki/methods/egsde]]；FreeDoM 把它换成 $E(\hat x_0,c)$ 的距离族，见 [[wiki/methods/freedom]]。这两条都是本页 §2–§4 的具体化。

## 关系

- 理论底座：[[wiki/concepts/score-sde]]（反向 SDE、条件反向 SDE 的贝叶斯分解）
- 引导家族：[[wiki/concepts/classifier-guidance]]（写法 A 的实例）、[[wiki/concepts/classifier-free-guidance]]（免外部打分器）、[[wiki/concepts/energy-guidance]]（任意能量 + PoE）、[[wiki/concepts/training-free-guidance]]（点估计 + 现成模型，免重训）
- 方法实例：[[wiki/methods/egsde]]、[[wiki/methods/freedom]]
- 待补页：DPS、Tweedie/Empirical-Bayes 独立概念页
