---
type: source
title: "Denoising Diffusion Bridge Models (DDBM)"
aliases: [DDBM, Denoising Diffusion Bridge Models, "Zhou et al. 2023", 扩散桥模型, diffusion bridge]
tags: [diffusion, diffusion-bridge, image-translation, score-based, doob-h-transform, flow-matching]
status: active
created: 2026-06-01
updated: 2026-06-01
raw: "[[raw/literature-notes/zhouDenoisingDiffusionBridge2023]]"
authors: [Linqi Zhou, Aaron Lou, Samar Khanna, Stefano Ermon]
venue: "ICLR 2024 (arXiv 2309.16948)"
year: 2023
arxiv: "2309.16948"
---

# Denoising Diffusion Bridge Models (DDBM)

> 文献笔记：[[raw/literature-notes/zhouDenoisingDiffusionBridge2023]] · arXiv [2309.16948](http://arxiv.org/abs/2309.16948) · Zhou, Lou, Khanna & Ermon 2023
> 数学背书（用户手写推导）：[[raw/notes/生成元方法对于SDE]] —— 已提炼为 [[wiki/concepts/infinitesimal-generator]] / [[wiki/concepts/doob-h-transform]]

## 一句话

不再 noise→data，而是在**两个配对端点之间架一座随机桥** $x_0(\text{data})\leftrightarrow x_T=y(\text{condition})$：用 [[wiki/concepts/doob-h-transform|Doob's h-transform]] 把任意 VP/VE 扩散过程**钉死终点** $x_T=y$ 得到 forward bridge SDE，再学桥的 score $s=\nabla_{x_t}\log q(x_t\mid x_T)$ 沿反向 SDE / [[wiki/concepts/probability-flow-ode|PF-ODE]] 从 $y$ 还原 $x_0$。把源端设成纯噪声即**严格退化回** [[wiki/concepts/score-sde|score-based diffusion]]；在 image-to-image translation 上大幅超越 baseline（Edges→Handbags FID **1.83**），无条件生成与 EDM 持平（CIFAR-10 FID **2.06**）。

## Motivation

[[wiki/concepts/diffusion-process|扩散模型]]把 noise→data 做得很好，但**很多任务的输入不是随机噪声**（image editing / translation / restoration 的输入是另一张图）。要把这种"成对的、确定的端点信息"塞进扩散，过去只能靠 guidance、projected sampling、或 [[wiki/methods/sdedit|SDEdit]] 式加噪——都属于**外挂**，没有把"两端分布之间的传输"作为一等公民来建模。

DDBM 要的是一个**原生**的范式：直接对"端点 $x_0$ ↔ 端点 $x_T=y$"的桥过程建模，而且**复用扩散社区现成的设计**（VP/VE schedule、EDM preconditioning、higher-order sampler），不另起炉灶。

## Method

![[zhouDenoisingDiffusionBridge2023-1780285042493.webp]]

> 图：上=forward bridge SDE（公式 5），漂移多出 Doob h 项 $\mathbf h=\nabla_{x_t}\log p(x_T\mid x_t)\big|_{x_T=y}$ 把过程拉向终点；中=tiger→corgi 的 translation 桥（中间是含噪插值）；下=反向 PF-ODE（公式 7），唯一要学的是 score $\mathbf s=\nabla_{x_t}\log q(x_t\mid x_T)\big|_{x_T=y}$。

### 1. 第一层：选一个普通扩散过程

选 VP 或 VE，给出转移核 $p(x_t\mid x_0)=\mathcal N(\alpha_t x_0,\sigma_t^2 I)$（VP）或 $\mathcal N(x_0,\sigma_t^2 I)$（VE），由此定 $f(x_t,t)$ 与 $g^2(t)$。记 $\mathrm{SNR}_t=\alpha_t^2/\sigma_t^2$。

### 2. 第二层：Doob h-transform 钉死终点 → forward bridge SDE

令 $h(x,t)=p(X_T=y\mid X_t=x)$，[[wiki/concepts/doob-h-transform|h-transform]] 在原漂移上加一个"终点吸引项"：
$$
\mathrm dx_t=\big[\,f(x_t,t)+g^2(t)\,\underbrace{\nabla_{x_t}\log p(x_T\mid x_t)}_{h(x_t,t,y,T)}\big]\mathrm dt+g(t)\,\mathrm dw_t,\quad x_0\sim q_{\text{data}},\ x_T=y. \tag{5}
$$
$h$ 项保证样本**一定到达** $x_T=y$。这一步的严格推导（生成元 → Kolmogorov backward → $\mathcal L^h=\tfrac1h\mathcal L(h\cdot)-\tfrac{\cdot}{h}\mathcal L h$ 展开出 $g^2\nabla\log h$）见 [[wiki/concepts/infinitesimal-generator]]。

### 3. 第三层：可解析的桥采样分布（训练标签来源）

定义 $q(x_t\mid x_0,x_T):=p(x_t\mid x_0,x_T)$，由 Bayes $\propto p(x_T\mid x_t)\,p(x_t\mid x_0)$（两项皆高斯）得闭式高斯（公式 8）：
$$
\hat\mu_t=\frac{\mathrm{SNR}_T}{\mathrm{SNR}_t}\frac{\alpha_t}{\alpha_T}x_T+\alpha_t x_0\Big(1-\frac{\mathrm{SNR}_T}{\mathrm{SNR}_t}\Big),\qquad
\hat\sigma_t^2=\sigma_t^2\Big(1-\frac{\mathrm{SNR}_T}{\mathrm{SNR}_t}\Big).
$$

### 4. 反向：唯一未知是 score（Theorem 1）

条件过程 $q(x_t\mid x_T)$ 的时间反演给出反向 SDE（6）与 probability-flow ODE（7）：
$$
\mathrm dx_t=\Big[f-g^2\big(\tfrac12 s(x_t,t,y,T)-h(x_t,t,y,T)\big)\Big]\mathrm dt,\quad x_T=y, \tag{7}
$$
其中 $s(x,t,y,T)=\nabla_{x_t}\log q(x_t\mid x_T)$ 是**全网络唯一要学的量**（$f,g,h$ 都由所选扩散过程解析给出）。

### 5. 训练：denoising bridge score matching

$$
\mathcal L(\theta)=\mathbb E_{x_t,x_0,x_T,t}\Big[w(t)\big\|s_\theta(x_t,x_T,t)-\nabla_{x_t}\log q(x_t\mid x_0,x_T)\big\|^2\Big] \tag{9}
$$
回归**条件 score**（闭式，来自第 3 步），最优解满足 $s_\theta=\nabla_{x_t}\log q(x_t\mid x_T)$——与 [[wiki/concepts/score-matching|DSM]]「用条件 score 当监督却学到边缘 score」是同一套路。配 EDM 风格的 **score reparameterization**（公式 10，经 pred-$x$ 网络 $D_\theta$）。

### 6. 采样：hybrid sampler（Algorithm 1）

higher-order ODE（Heun，来自 EDM / Karras et al. 2022）+ 在步间插入 **scheduled Euler-Maruyama** SDE 步（[[wiki/concepts/predictor-corrector-sampling|PC]] 思想），由 step ratio $s$ 控制注入多少随机性、guidance strength $w$ 放大 $h$ 项。**纯 ODE（$s=0$）会糊**（确定性反向 ODE 给"平均"路径）；加随机性后 FID 显著下降（Edges→Handbags 最优在 $s\approx0.3$）——这点与"反向桥需要 stochasticity 保多样性"一致。

## Results

### Image-to-image translation（pixel space，Table 2；同架构、N=40 NFE）

| 方法 | Handbags FID↓ | Handbags LPIPS↓ | DIODE FID↓ | DIODE LPIPS↓ |
|---|---|---|---|---|
| Pix2Pix | 74.8 | 0.356 | 82.4 | 0.556 |
| DDIB | 186.84 | 0.869 | 242.3 | 0.798 |
| [[wiki/methods/sdedit\|SDEdit]] | 26.5 | 0.271 | 31.14 | 0.714 |
| [[wiki/methods/rectified-flow\|Rectified Flow]] | 25.3 | 0.241 | 77.18 | 0.534 |
| I²SB | 7.43 | 0.244 | 9.34 | 0.373 |
| **DDBM (VE)** | 2.93 | 0.131 | 8.51 | 0.226 |
| **DDBM (VP)** | **1.83** | 0.142 | **4.43** | 0.244 |

- **DDBM(VP) 在两个数据集 FID 都断层第一**；I²SB 是最接近的 baseline，但低 NFE 下落后。
- 🔴 **关键观察（对你 thesis 有用）**：Rectified Flow 这种 OT-based 方法在**两域低层相似度低**时崩盘（DIODE 77.18 vs Handbags 25.3）——OT 直线传输假设"端点几何接近"，跨域差异大时失效；DDBM 的随机桥不依赖此假设。

### 无条件生成（源端=高斯，Table 4；FID on 50K）

| 方法 | CIFAR-10 NFE / FID↓ | FFHQ-64 NFE / FID↓ |
|---|---|---|
| DDPM | 1000 / 3.17 | 1000 / 3.52 |
| [[wiki/methods/rectified-flow\|Rectified Flow]] | 127 / 2.58 | 152 / 4.45 |
| EDM | 35 / 2.04 | 79 / 2.53 |
| **DDBM** | 35 / **2.06** | 79 / **2.44** |

- 源端设成高斯时 DDBM 退化回扩散，**与 EDM 几乎持平**（CIFAR 2.06 vs 2.04，FFHQ 反而 2.44 < 2.53）——印证"为更一般任务设计、却不牺牲生成"的卖点。

## 关系（与已有 wiki 的关联）

- **本页是新建概念/方法页的源**：[[wiki/methods/ddbm]]、[[wiki/concepts/doob-h-transform]]、[[wiki/concepts/infinitesimal-generator]]、[[wiki/concepts/diffusion-bridge]]、[[wiki/concepts/stochastic-interpolants]]（均由本次 ingest 创建）。
- **退化回 [[wiki/concepts/score-sde|Score SDE]]（§6.1 Case 1，站得住的"统一"）**：源端 $x_T\sim\mathcal N$ 时，公式 (6)/(7) **严格**约化为普通扩散的反向 SDE / PF-ODE。这是 DDBM 真正包含 diffusion 的方向。
- **与 [[wiki/concepts/flow-matching|OT-Flow-Matching]] / [[wiki/methods/rectified-flow|Rectified Flow]] 的关系（§6.1 Case 2，话术多于数学）**：论文称"unifies OT-FM"，但正文写明这只在 **noiseless 极限 $c\to0$ + 特定 VE schedule**（公式 15）、且"with some additional caveat to handle additional input $x_T$"下成立——是**有条件的极限约化，不是严格特例**。三处不可消差异：随机 SDE vs 确定 ODE、Doob 钉端点 vs 不钉、paired vs independent coupling（详见 [[wiki/concepts/diffusion-bridge]] 对照表）。
- **真正的数学公约数是 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo & Vanden-Eijnden）**：论文自承 Albergo et al. 2023 "presents a general theory ... unifying flow and diffusion, and shows that a bridge can be constructed from both an ODE and SDE perspective"，并明确 **DDBM 用的是不同的 denoising bridge score-matching loss**。即「bridge SDE（DDBM）vs bridge ODE（flow）」的严格归宿在 stochastic interpolants，而非 DDBM 本身。
- **数学机制**：[[wiki/concepts/doob-h-transform]]（终点吸引漂移 $g^2\nabla\log h$ 的来源）、[[wiki/concepts/infinitesimal-generator]]（生成元 → Kolmogorov backward → [[wiki/concepts/fokker-planck-equation|Fokker-Planck]] → h-transform 生成元，对应 [[raw/notes/生成元方法对于SDE]]）。
- **训练 / 采样复用**：[[wiki/concepts/score-matching]]（条件→边缘恒等式）、[[wiki/concepts/predictor-corrector-sampling]]（hybrid sampler 的 SDE corrector 步）、[[wiki/concepts/probability-flow-ode]]（公式 7 是桥的 PF-ODE）。
- **人物 / 机构**：[[wiki/entities/linqi-zhou]]、[[wiki/entities/aaron-lou]]、Samar Khanna、[[wiki/entities/stefano-ermon]]；[[wiki/entities/stanford]]。
- **评测**：[[wiki/benchmarks/cifar10]]、Edges→Handbags / DIODE-Outdoor（translation 数据集，暂未单列）。

## 对我的 thesis 的启示

<!-- 用户选择：本节留空，由用户自己写。以下仅列 Claude 识别到的候选角度（待你定，未写入 overview/research）。 -->

> 本节按你的指示**留给你自己写**，未改动 [[wiki/overview]] working thesis。下列是我在 ingest 时识别到的候选挂钩，仅供你取舍：

- 候选 ①（你 why-read 提到的 idea）：把 DDBM 改进为 **score-based bridge SDE**——在「bridge SDE 一侧、换插值路径/噪声 schedule」这个格子里做文章。
- 候选 ②（你红色批注）：原文"ODE 方法经验上打不过 diffusion"在 SD3/FLUX 时代已不成立；DDBM 的 Table 2 反而显示 **OT-ODE 在跨域 translation 上崩盘**（DIODE RF 77.18）——这是"flow vs bridge"在编辑/翻译任务上的一条可证伪边界。
- 候选 ③：DDBM 把 editing/translation 形式化为"两端点随机桥"，是 [[wiki/overview]] 推论 1「editing-as-transport」的 SDE 化身——但是否要纳入 working thesis 由你决定。

## Open questions / 待追

- [ ] 🔵 [[wiki/concepts/stochastic-interpolants|Stochastic Interpolants]]（Albergo & Vanden-Eijnden 2023）原文 ingest——这是"bridge SDE vs bridge ODE"的严格统一框架，wiki 现仅有 draft 占位页。
- [ ] I²SB（Liu et al. 2023，tractable Schrödinger Bridge）：最接近的 baseline，与 DDBM 的 loss / 构造差异。
- [ ] Schrödinger Bridge / Bridge-Matching（Shi et al. 2023，Iterative Markovian Fitting）线。
- [ ] EDM（Karras et al. 2022）preconditioning 与 higher-order sampler——DDBM 直接复用，wiki 暂无独立页。
- [ ] hybrid sampler 中 stochasticity（step ratio $s$）对 translation 多样性 vs 保真度的 trade-off，与推论 2「在哪个 $t$ 注入」的关系。
