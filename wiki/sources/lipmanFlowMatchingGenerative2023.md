---
type: source
title: "Flow Matching for Generative Modeling"
aliases: [Flow Matching, FM, "Lipman et al. 2023", CFM 原文, Conditional Flow Matching]
tags: [flow-matching, cnf, optimal-transport, generative-model, foundational]
status: stable
created: 2026-05-24
updated: 2026-05-24
raw: "[[raw/literature-notes/lipmanFlowMatchingGenerative2023]]"
authors: [Yaron Lipman, Ricky T. Q. Chen, Heli Ben-Hamu, Maximilian Nickel, Matt Le]
venue: ICLR 2023
year: 2023
arxiv: "2210.02747"
---

# Flow Matching for Generative Modeling

> 文献笔记：[[raw/literature-notes/lipmanFlowMatchingGenerative2023]] · arXiv [2210.02747](http://arxiv.org/abs/2210.02747) · Lipman et al. 2023 (ICLR)
> ⚠️ **时间约定与本 vault 的 diffusion 页相反**：本文 $t=0$ 是**噪声** $p_0=\mathcal N(0,I)$、$t=1$ 是**数据** $p_1=q$，生成沿 $t:0\to1$ **正向**积分 ODE。而 [[wiki/methods/ddpm|DDPM]]/[[wiki/concepts/score-sde|Score SDE]] 的前向 $t:0\to T$ 是数据→噪声。读关系段时注意方向。

## 一句话

把生成建模从「设计一条加噪 SDE」改写成「**直接选一条 probability path $p_t$（噪声↔数据）并用网络回归它的 velocity field $v_t$**」，沿 ODE $\dot x=v_t(x)$ 生成——这就是 simulation-free 地训练 [[wiki/concepts/continuous-normalizing-flow|CNF]]。核心技巧 [[wiki/concepts/conditional-flow-matching|Conditional Flow Matching (CFM)]] 用 per-example 的条件目标替掉 intractable 的边缘目标、**梯度完全相同**（与 [[wiki/concepts/score-matching|DSM]] 同一套路）。Gaussian 路径族**把 diffusion 路径收为特例**；而新引入的 [[wiki/concepts/optimal-transport-path|Optimal Transport (OT) 路径]]给出**直线匀速**轨迹，训练/采样更快、泛化更好。

## Motivation

[[wiki/concepts/continuous-normalizing-flow|CNF]]（Chen et al. 2018）表达力强，但传统训练要在前向/反传中**反复数值积分 ODE**（simulation-based），昂贵且难 scale。作者要的是：

- **simulation-free** 地训练 CNF（训练时不解 ODE）；
- 摆脱 diffusion 对「先构造一个加噪过程、再间接定义路径」的依赖，**直接对 probability path 本身做设计**；
- 由此解锁 diffusion 之外的路径——尤其是 **OT 直线路径**，更高效。

🔴 用户 takeaway（全文灵魂）：**FM 把问题从"如何设计一个 SDE"转成"如何设计一条好的 probability path，并学它的 velocity field"**。关于 $p(x)$ 的全部知识被装进 $v(x,t)$ 里——这正是与 score 版的根本分工差异（见下）。

## Method

### 0. 背景：CNF、push-forward、连续性方程（采纳用户 takeaway #1–2）

CNF 用一个含时向量场定义确定性流：
$$\frac{d}{dt}\phi_t(x)=v_t(\phi_t(x)),\qquad \phi_0(x)=x.$$
流把基分布 push-forward 成 $p_t=[\phi_t]_*p_0$，密度满足换元式（原密度 × 体积压缩系数）
$$p_t(x)=p_0(\phi_t^{-1}(x))\,\Big|\det\tfrac{\partial\phi_t^{-1}}{\partial x}(x)\Big|.$$
判断 $(p_t,v_t)$ 是否自洽，更方便的是查**连续性方程**（这是 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]] 在**无扩散项**$g=0$ 时的退化——FM 是确定性流，没有 $\tfrac12 g^2\Delta p$ 项）：
$$\frac{\partial p_t(x)}{\partial t}+\nabla\!\cdot\!\big(p_t(x)\,v_t(x)\big)=0.$$

> 与 score 版的分工（⚫ 用户批注）：SDE 里 $v$ 解析已知、要学的是 $\nabla\log p$；CNF 里 $v$ 未知且复杂，**关于 $p(x)$ 的知识全部含于 $v(x,t)$**。

### 1. Flow Matching 目标与它的 intractability

给定想要的边缘路径 $p_t$（$p_0=$ 噪声，$p_1\approx q$ 数据）及其生成场 $u_t$，FM 直接回归：
$$\mathcal L_{\mathrm{FM}}(\theta)=\mathbb E_{t,\,p_t(x)}\big\|v_t(x;\theta)-u_t(x)\big\|^2.\tag{5}$$
问题：**边缘** $u_t,p_t$ 都没有闭式、不可直接采样，目标 intractable。

### 2. CFM：per-example 构造 + 梯度等价（核心，采纳用户 takeaway #3）

用 per-sample 的**条件**路径/场聚合出边缘量：
$$p_t(x)=\int p_t(x\mid x_1)\,q(x_1)\,dx_1,\qquad
u_t(x)=\int u_t(x\mid x_1)\,\frac{p_t(x\mid x_1)\,q(x_1)}{p_t(x)}\,dx_1.\tag{6,8}$$
（用户类比流体 $\bar v=\overline{\rho v}/\bar\rho$，且用连续性方程可证「边缘场生成边缘路径」。）**Conditional Flow Matching** 改回归条件场：
$$\mathcal L_{\mathrm{CFM}}(\theta)=\mathbb E_{t,\,q(x_1),\,p_t(x\mid x_1)}\big\|v_t(x;\theta)-u_t(x\mid x_1)\big\|^2.\tag{9}$$
**Theorem 2**：当 $p_t(x)>0$，$\mathcal L_{\mathrm{FM}}$ 与 $\mathcal L_{\mathrm{CFM}}$ 至多差一个与 $\theta$ 无关的常数，故 $\nabla_\theta\mathcal L_{\mathrm{FM}}=\nabla_\theta\mathcal L_{\mathrm{CFM}}$。证明就是把 L2 按 $x$ 展开，交叉项里的 $\mathbb E_{x_1\mid x,t}[u_t(x\mid x_1)]=u_t(x)$（最优解是条件期望=边缘场）——**与 [[wiki/concepts/score-matching|DSM]]「用条件 score 当监督却学到边缘 score」是同一个数学套路**。论文 §5 亦明言 CFM "draws inspiration from" Vincent 2011 的 DSM，只是把对象从 score 换成 vector field。

### 3. 高斯条件路径：可操作配方（采纳用户 takeaway #4）

取
$$p_t(x\mid x_1)=\mathcal N\big(x\mid \mu_t(x_1),\,\sigma_t(x_1)^2 I\big),\tag{10}$$
边界 $\mu_0=0,\sigma_0=1$（$p_0=\mathcal N(0,I)$）、$\mu_1=x_1,\sigma_1=\sigma_{\min}$（$p_1$ 集中在 $x_1$）。对应流与条件场：
$$\psi_t(x_0)=\sigma_t(x_1)\,x_0+\mu_t(x_1),\qquad
u_t(x\mid x_1)=\frac{\sigma_t'(x_1)}{\sigma_t(x_1)}\big(x-\mu_t(x_1)\big)+\mu_t'(x_1).\tag{11,15}$$
**设计好 $\mu_t,\sigma_t$ 即可互推**。diffusion 路径（VP/VE）只是 $\mu_t,\sigma_t$ 的某种选择，轨迹**弯曲**。

### 4. OT 路径：直线匀速（采纳用户 takeaway #5）

取 $\mu_t(x_1)=t\,x_1$、$\sigma_t(x_1)=1-(1-\sigma_{\min})t$，则
$$x_t=\big(1-(1-\sigma_{\min})t\big)x_0+t\,x_1,\qquad
u_t(x\mid x_1)=\frac{x_1-(1-\sigma_{\min})x}{1-(1-\sigma_{\min})t}.$$
代入 $x=x_t$ 得**恒定方向与大小** $x_1-(1-\sigma_{\min})x_0$——这正是两个高斯之间的 **Optimal Transport displacement map**。相比 diffusion 路径"只在末段才去掉噪声"，OT 路径**近似线性**地从噪声走到数据。

### 5. 算法流程（采纳用户 takeaway #6）

1. 采数据 $x_1\sim q$、噪声 $x_0\sim\mathcal N(0,I)$、时间 $t\sim U[0,1]$；
2. 构造中间点 $x_t=\psi_t(x_0)=\mu_t(x_1)+\sigma_t(x_1)x_0$；
3. 用条件场 $u_t(x_t\mid x_1)$ 作标签，回归 $\|v_t(x_t;\theta)-u_t(x_t\mid x_1)\|^2$；
4. 采样：$x_0\sim\mathcal N(0,I)$，用现成 ODE solver（默认 `dopri5`，tol 1e-5）正向解 $\dot x=v_t(x;\theta)$ 到 $t=1$。

> 采样直觉（用户 takeaway #7）：初始点更靠近"dog"时其条件路径权重大、"cat"权重小，边缘场把样本逐渐带向"dog"。

## Results

**Table 1（同一 U-Net 架构，Dhariwal & Nichol 2021；NLL=bits/dim↓，FID↓，NFE=adaptive solver 平均函数求值↓，unconditional）**

| Model | CIFAR-10 | ImageNet-32 | ImageNet-64 |
|---|---|---|---|
| [[wiki/methods/ddpm\|DDPM]] | 3.12 / 7.48 / 274 | 3.54 / 6.99 / 262 | 3.32 / 17.36 / 264 |
| Score Matching | 3.16 / 19.94 / 242 | 3.56 / 5.68 / 178 | 3.40 / 19.74 / 441 |
| ScoreFlow | 3.09 / 20.78 / 428 | 3.55 / 14.14 / 195 | 3.36 / 24.95 / 601 |
| FM w/ Diffusion | 3.10 / 8.06 / 183 | 3.54 / 6.37 / 183 | 3.33 / 16.88 / 187 |
| **FM w/ OT** | **2.99 / 6.35 / 142** | **3.53 / 5.02 / 122** | **3.31 / 14.45 / 138** |

- **FM-OT 在三项指标上一致最好**；同架构下"FM w/ Diffusion"已比 DDPM 略稳、NFE 更低，换 OT 路径再上一台阶。
- **ImageNet-128×128**：FM-OT FID **20.9**、NLL 2.90，是当时 SOTA（仅次于用 self-sup 条件的 IC-GAN）。
- **更快训练**：ImageNet-128 上 Dhariwal & Nichol 训 4.36M iters（batch 256），FM 仅 500k iters（batch 1.5k，模型大 25%）即≈33% 更少图像吞吐就收敛（Fig 5）。
- **更省采样**：FM-OT 达到同等数值误差只需约 **60% 的 NFE**；且 score matching 的采样成本随训练增长，FM 则**恒定**。
- **条件超分（64→256，ImageNet val，Table 2）**：FM-OT FID **3.4** / IS **200.8**，优于 SR3（5.2 / 180.1）；PSNR/SSIM 与 SR3 相当。

## 关系（与已有 wiki 的关联）

- **本页是新建概念页的源**：[[wiki/concepts/flow-matching]]、[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/continuous-normalizing-flow]]、[[wiki/concepts/optimal-transport-path]]（均由本次 ingest 创建）。
- **与 [[wiki/concepts/score-sde|Score SDE]] / [[wiki/concepts/probability-flow-ode|probability-flow ODE]]**：PF-ODE 也是"生成同一族 $p_t$ 的确定性 ODE"，但它是**先训 score、事后导出** ODE；FM **直接把训练目标换成回归速度场**，ODE 是训出来的本体。一句话延续 vault 既有提法：**DDIM/PF-ODE = diffusion 的训练 + flow 的采样；FM = 连训练也 flow 化**。
- **把 diffusion 收为特例**：[[wiki/methods/ddpm|DDPM]]（VP）/[[wiki/methods/ncsn|NCSN]]（VE）的高斯路径是 Gaussian 路径族的成员；FM w/ Diffusion 复现其结果但训练更稳。
- **CFM ≡ DSM 的 vector-field 版**：[[wiki/concepts/score-matching]] 的条件→边缘恒等式与本文 Theorem 2 同构。
- **连续性方程 = 无扩散的 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]]**。
- **下游 / 并行**：[[wiki/methods/rectified-flow|Rectified Flow]]（Liu et al. 2022）、Stochastic Interpolants（Albergo & Vanden-Eijnden 2022）为并行工作；FM-OT 是 SD3 / FLUX 一线 rectified-flow 训练目标的直接源头（待 ingest 原文）。
- 人物 / 机构：[[wiki/entities/yaron-lipman]]、[[wiki/entities/ricky-chen]]（含 Heli Ben-Hamu、Maximilian Nickel、Matt Le）；[[wiki/entities/meta-ai-fair]]。
- 评测：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/imagenet]]。

## 对我的 thesis 的启示

- **直接命中 overview 推论 1（可变性光谱）的"训练目标"最贵一档**：FM **真的换了训练目标**（回归速度场而非 ε / score），却仍落在"迭代生成 + 网络预测速度场 + 沿生成链注入条件"这条不变范式里——是"训练目标可演化、范式不变"的最干净样本，可关闭多处"待 ingest FM 原文"。
- **强化推论 3（采样加速）**：OT 直线路径让 NFE 显著下降、采样成本训练期恒定——对反复采样的编辑场景价值直接。
- **对推论 2（介入时间步）**：FM 的 $t$ 与 diffusion 的 $t$ **方向相反且语义不同**（这里 $t$ 是噪声→数据的插值系数），讨论"在哪个时间步注入条件"时必须先统一坐标，否则跨范式比较会错位——这是 thesis 里要立的一个 caveat。

> 拟据此微调 [[wiki/overview]] working thesis（推论 1/3 升级、补 FM 时间约定 caveat），已单独做成 diff 待用户确认，未直接改动。

## Open questions / 待追

- [ ] FM-OT → Rectified Flow → SD3 / FLUX 的演化链：reflow、few-step、与本文 OT 路径的精确异同（待 ingest Liu et al. 2022 / SD3 / FLUX）。
- [ ] 在 **text-guided editing** 上，FM 模型的 inversion 怎么做？OT 直线路径是否让 inversion 往返更易闭合（对照 [[wiki/methods/ddim]] failure mode）？
- [ ] 非各向同性高斯 / 更一般 kernel 的路径（作者在 Conclusion 点名的未来方向）。
- [ ] FM 的 $t$ 反向约定在跨范式（与 score-based）比较时的统一记法。
