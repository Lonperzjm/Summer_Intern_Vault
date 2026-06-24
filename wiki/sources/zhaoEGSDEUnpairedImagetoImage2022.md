---
type: source
title: "EGSDE: Unpaired Image-to-Image Translation via Energy-Guided Stochastic Differential Equations"
aliases: [EGSDE, "Zhao et al. 2022", energy-guided SDE]
tags: [diffusion, score-sde, guidance, energy-guidance, unpaired-i2i, translation, training-free-generator]
status: stable
created: 2026-06-21
updated: 2026-06-21
raw: "[[raw/literature-notes/zhaoEGSDEUnpairedImagetoImage2022]]"
authors: [Min Zhao, Fan Bao, Chongxuan Li, Jun Zhu]
venue: NeurIPS 2022
year: 2022
arxiv: "2207.06635"
---

# EGSDE: Unpaired Image-to-Image Translation via Energy-Guided Stochastic Differential Equations

> 文献笔记：[[raw/literature-notes/zhaoEGSDEUnpairedImagetoImage2022]] · arXiv [2207.06635](http://arxiv.org/abs/2207.06635) · Zhao, Bao, Li, Zhu · NeurIPS 2022 · code [ML-GSAI/EGSDE](https://github.com/ML-GSAI/EGSDE)
> 本页是师兄 6/2 推的"discriminative logits → energy → guidance"范式的奠基样本，直接服务 [[research/ideas]] 顶部 energy-guidance 候选条。

## 一句话

只在**目标域**训练一个 [[wiki/concepts/score-sde|score SDE]]，反向采样时叠加一个由**源/目标双域**预训练的 energy 引导项——energy 让输出"丢掉源域特定特征（realism）、保留域无关结构（faithfulness）"。本质是把 [[wiki/concepts/classifier-guidance|classifier guidance]] 从"单分类器梯度"推广到"任意能量函数 + product-of-experts"，且全程**冻结生成器**。

## Motivation

unpaired I2I（无配对图像翻译，如 Cat→Dog）的 SBDM 路线（[[wiki/methods/sdedit|SDEdit]]、ILVR）做法：把源图加噪到中间时刻当反向起点，再用目标域 score 去噪。作者点出的痛点（🟡 p.3）：

> **现有方法完全忽略了源域的训练数据**——源图只当了反向采样的起点，反向过程里源信息再不参与，导致 unpaired I2I 次优。

EGSDE 的动机：用一个在**源域 + 目标域**上预训练的 energy function，让源信息**全程**以梯度形式参与反向 SDE，从而同时拿到 realism 与 faithfulness，并提供二者之间的可调旋钮。

## Method

### 1. 能量引导的反向 SDE（🟡 p.3 eq6）

记被生成（去噪中）的样本为 $y$、源图为 $x_0$。起点 $y_M\sim q_{M|0}(y_M\mid x_0)$（把源图加噪到 $M=0.5T$，🟡 p.4），随后跑能量引导的反向 SDE：

$$\mathrm dy=\Big[f(y,t)-g(t)^2\big(\,\underbrace{s(y,t)}_{\text{目标域 score}}-\nabla_y\mathcal E(y,x_0,t)\big)\Big]\mathrm dt+g(t)\,\mathrm d\bar w.$$

等价地，引导后的 score（my-summary #1）：
$$s_{\text{EGSDE}}(y_t,t,x_0)=s_{\mathcal Y}(y_t,t)-\nabla_{y_t}\mathcal E(y_t,x_0,t).$$
🟡 p.2：EGSDE 通过这条把 energy 与预训练 SDE 复合的反向 SDE，**定义了一个合法的条件分布**——不是 ad-hoc 加项。

### 2. 能量的两个专家（🟡 p.4 eq8）

$$\mathcal E(y,x_0,t)=\lambda_s\,\mathbb E_{q_{t|0}(x_t\mid x_0)}\big[S_s(y_t,x_t,t)\big]\;-\;\lambda_i\,\mathbb E_{q_{t|0}(x_t\mid x_0)}\big[S_i(y_t,x_t,t)\big].$$

- **域特定专家 $S_s$（realism，🟡 p.4–5 eq9）**：$E_s$ 是"一个在两域上训练、判别图来自源域还是目标域的**分类器的除最后一层外的全部**"（🟡 p.4，**时间相关 / noise-aware**）。$S_s$ 取生成样本 $y_t$ 与噪化源 $x_t$ 的域特定特征**余弦相似度**：
$$S_s(y_t,x_t,t)=\tfrac1{HW}\sum_{h,w}\frac{E_s^{hw}(x_t,t)^\top E_s^{hw}(y_t,t)}{\lVert E_s^{hw}(x_t,t)\rVert\,\lVert E_s^{hw}(y_t,t)\rVert}.$$
能量里带 $+\lambda_s S_s$ → 最小化能量 = **降低**与源的域特定相似度 → 抹掉源域专有属性 → realism。
- **域无关专家 $S_i$（faithfulness，🟡 p.5 eq10 / p.9）**：$E_i$ 是一个**低通滤波器**，取负 L2：
$$S_i(y_t,x_t,t)=-\lVert E_i(y_t,t)-E_i(x_t,t)\rVert^2.$$
能量里带 $-\lambda_i S_i=+\lambda_i\lVert\cdot\rVert^2$ → 最小化能量 = **拉近**低通特征 → 保结构/颜色/背景 → faithfulness。

### 3. ⭐ 关键设计：noisy-aligned，不是 clean-estimate（🟢 用户 p.4 批注 + my-summary #2）

EGSDE **故意不**先从 $y_t$ 估 $\hat y_0$ 再评分，而是把源图 $x_0$ **加噪到同一个 $t$** 得 $x_t$，做 **noisy-to-noisy** 比较：
$$x_0\xrightarrow{\text{加噪到 }t} x_t,\qquad (y_t,x_t,t)\to\mathcal E\to\nabla_{y_t}\mathcal E.$$
用户批注点破其理由：「避免把 noisy $y_t$ 直接和 clean $x_0$ 比——二者噪声水平不同，特征相似性可能没意义」。**代价**：$E_s$ 必须在带噪图上训成时间相关分类器（每个 task 重训），无法直接复用现成 clean-domain 判别器。这正是它与"Tweedie/一步 $\hat x_0$ + 现成模型"训练-free 引导线（FreeDoM / UGD / DPS / [[wiki/concepts/classifier-guidance|classifier guidance]]）的**相反取舍**——见「对我的 thesis 的启示」。

### 4. Product of Experts 解释（🟡 p.1 / my-summary #3）

$$\tilde p(y_t\mid x_0)\propto \underbrace{p_{r1}(y_t\mid x_0)}_{\text{目标域 SDE/SDEdit}}\;\underbrace{p_{r2}(y_t\mid x_0)}_{\text{丢源域特定}}\;\underbrace{p_f(y_t\mid x_0)}_{\text{保域无关}}.$$
三个专家各只贡献 realism 或 faithfulness；这解释了为什么 energy 梯度会作为反向 SDE 的 **drift 修正项**出现。离散采样落到一个高斯转移（🟡 p.6 eq15）：$\tilde p(y_t\mid y_s)\approx\mathcal N\big(\mu(y_s,h)-\Sigma(s,h)\nabla E,\ \Sigma(s,h)I\big)$——energy 梯度平移转移分布的均值。

## Results

- **AFHQ**（[[wiki/benchmarks/afhq|AFHQ]]）：调超参后 **Cat→Dog FID 51.04**、**Wild→Dog FID 50.43**（🟡 abstract）。
- **CelebA-HQ Male→Female**：第三个任务（具体数见原文）。
- **三任务 × 四指标**：realism（FID）+ faithfulness（如 L2 / PSNR / SSIM）。结论：在几乎所有设置上**优于现有 SBDM 路线**（SDEdit、ILVR、CycleGAN 系等），拿到 SOTA realism **且不损 faithfulness**。
- **$\lambda_s,\lambda_i$ 给出 realism↔faithfulness 的连续旋钮**——和 [[wiki/methods/sdedit|SDEdit]] 的单旋钮 $t_0$ 类似，但这里是两路能量分别可调，解耦度更高。

## 关系

- **底座 / 直接基线**：[[wiki/methods/sdedit|SDEdit]]——EGSDE 的 $p_{r1}$ 专家就是 SDEdit 式"加噪源图 + 目标域 reverse"；EGSDE = SDEdit + 两路 energy 引导。理论根在 [[wiki/concepts/score-sde|Score SDE]] 的条件反向 SDE。
- **概念归属**：[[wiki/concepts/energy-guidance|energy guidance]]（本页催生的新概念页）；是 [[wiki/concepts/classifier-guidance|classifier guidance]] 的能量化推广（单分类器梯度 → 任意能量 + PoE）。
- **方法页**：[[wiki/methods/egsde|EGSDE]]。
- **与条件注入家族**：相对 [[wiki/methods/controlnet|ControlNet]]（改网络 + 再训练）/ [[wiki/concepts/in-context-conditioning|in-context]]（拼 token），EGSDE 走**冻结生成器 + 采样期梯度引导**这一路。
- **与 bridge 路线对照**：[[wiki/methods/ddbm|DDBM]] 改 SDE 端点来钉条件；EGSDE 不动端点、靠 energy 引导项注入条件——是"条件注入"的另一格。
- **作者 / 机构**：[[wiki/entities/jun-zhu|Jun Zhu]]（资深作者）、Fan Bao、Chongxuan Li、Min Zhao；[[wiki/entities/tsinghua-university|Tsinghua]] TSAIL/thu-ml。

## 对我的 thesis 的启示

> 🟣 用户 my-summary #4 是本篇与 thesis 的直接接口，逐字流入此节（也是 [[research/ideas]] energy-guidance 候选的核心假设）。

- **EGSDE 给了一个已验证有效的 energy-guided SDE 框架**，但两处局限是后续施力点：(i) faithful 专家**只是低通滤波器**（弱）；(ii) energy 作用在 **noisy state** 上、且 $E_s$ 须 noise-aware 重训。
- **自然后续（用户原创 framing）**：把 noisy-aligned 的 $\mathcal E(y_t,x_t,t)$ 换成 **clean-estimate-level** 能量
$$\widetilde{\mathcal E}(y_t,x_0,t)=E\big(\hat y_0(y_t,t),\,x_0\big),\qquad \nabla_{y_t}\widetilde{\mathcal E}=\Big(\tfrac{\partial \hat y_0}{\partial y_t}\Big)^{\!\top}\nabla_{\hat y_0}E,$$
并研究如何**高效稳定**地算这个雅可比-向量积。可迁移到 latent diffusion、[[wiki/concepts/flow-matching|Flow Matching]]、[[wiki/methods/rectified-flow|Rectified Flow]]、[[wiki/methods/flux-kontext|FLUX Kontext]]。
- **要正面回答的张力**：EGSDE 是**故意避开** clean-estimate 的（noisy-to-noisy 更稳）。换回 $\hat y_0$ 凭什么划算？判断：胜在**解耦/复用**——clean-estimate 让你复用任意现成判别器/reward（零重训），而 RF/FM 的 $\hat y_0=y_t-t\,v_\theta$ 因轨迹近直线更干净 → 高噪声下梯度更稳。**这条"clean-estimate 在 flow 上凭什么赢 diffusion"的假设须独立 sweep 验证/证伪**（红海纪律，见 [[research/ideas]]）。
- **对 [[wiki/overview]] working thesis 的影响**：thesis 现处 🔁 方向复审中。EGSDE 不改动 v0.1 的 bridge-SDE 押注，但为候选新方向（energy-guidance）提供了奠基支点——是否升级 thesis 取决于 sweep 结果，**本次 ingest 不动 overview 的 working thesis 版本号**。

## 我的 takeaways（讨论确认）

1. EGSDE = 冻结目标域 SDE + 双域 energy 引导；energy 梯度当反向 SDE 的 drift 修正。
2. 两专家：$E_s$（域分类器中间层，余弦相似度，降源域特定 → realism）+ $E_i$（低通滤波，负 L2，保域无关 → faithfulness）。
3. **noisy-aligned 是核心设计**：噪化源图做 noisy-to-noisy 比较，刻意不走 clean-estimate；代价是 $E_s$ 须 noise-aware 重训。
4. PoE 解释说明 energy 梯度为何以 drift 修正形式出现。
5. thesis 接口：把 noisy-aligned 换成 clean-estimate $E(\hat y_0,x_0)$ 并搬到 flow ——师兄圈的 novelty 点，但须 sweep。

## Open questions / 待追（🔵）

- [ ] ingest Dhariwal & Nichol 2021（classifier guidance 原型，$E_s$ 的近亲）
- [x] training-free / clean-estimate 引导全家桶：[[wiki/methods/freedom|FreeDoM]] ✅（已 ingest）、Universal Guidance、DPS、LGD、[TFG (2403.12404)](https://arxiv.org/abs/2403.12404)、[TFG-Flow (2501.14216)](https://arxiv.org/pdf/2501.14216) —— 与 EGSDE 的 noisy-aligned 取舍正面对比
- [ ] $\partial\hat y_0/\partial y_t$ 在 RF/FM 下的闭式与计算成本（clean-estimate 引导可行性的关键）
- [ ] ILVR / CycleGAN 系 unpaired I2I 基线补页
