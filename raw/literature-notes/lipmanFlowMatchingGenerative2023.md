---
type: literature-note
citekey: lipmanFlowMatchingGenerative2023
title: Flow Matching for Generative Modeling
aliases:
  - "@lipmanFlowMatchingGenerative2023"
authors: Yaron Lipman, Ricky T. Q. Chen, Heli Ben-Hamu, Maximilian Nickel, Matt Le
firstAuthor: Lipman
year: 2023
itemType: preprint
doi: 10.48550/arXiv.2210.02747
url: http://arxiv.org/abs/2210.02747
zotero: zotero://select/library/items/KTJKK2QR
tags:
  - literature
  - todo
status: unread
priority: P1
my-rating:
created: 2026-05-24
updated: 2026-05-24
ingested_to_wiki: true
wiki_page: "[[wiki/sources/lipmanFlowMatchingGenerative2023]]"
---

# Flow Matching for Generative Modeling

> [!info] @lipmanFlowMatchingGenerative2023 · Lipman et al. · 2023
> [Open in Zotero](zotero://select/library/items/KTJKK2QR) · [DOI](https://doi.org/10.48550/arXiv.2210.02747) · [URL](http://arxiv.org/abs/2210.02747) · [PDF](file:///home/lonper/Zotero/storage/Q69J7C42/Lipman%20等%20-%202023%20-%20Flow%20Matching%20for%20Generative%20Modeling.pdf)

## Abstract

> [!abstract]- Click to expand
> We introduce a new paradigm for generative modeling built on Continuous Normalizing Flows (CNFs), allowing us to train CNFs at unprecedented scale. Specifically, we present the notion of Flow Matching (FM), a simulation-free approach for training CNFs based on regressing vector fields of fixed conditional probability paths. Flow Matching is compatible with a general family of Gaussian probability paths for transforming between noise and data samples -- which subsumes existing diffusion paths as specific instances. Interestingly, we find that employing FM with diffusion paths results in a more robust and stable alternative for training diffusion models. Furthermore, Flow Matching opens the door to training CNFs with other, non-diffusion probability paths. An instance of particular interest is using Optimal Transport (OT) displacement interpolation to define the conditional probability paths. These paths are more efficient than diffusion paths, provide faster training and sampling, and result in better generalization. Training CNFs using Flow Matching on ImageNet leads to consistently better performance than alternative diffusion-based methods in terms of both likelihood and sample quality, and allows fast and reliable sample generation using off-the-shelf numerical ODE solvers.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 来看看flowmatching和diffusion（score版）的联系与区别吧
- 最好还能知道优劣与原因
- 最好还能读懂设计思路
%% end why-read %%

## 高亮颜色约定（个人 convention）

> 🟡 **Yellow** = 关键论点 / takeaway
> 🔴 **Red** = 我有异议 / 可疑结论 / 论文改进点
> 🟢 **Green** = 可借鉴的方法 / 公式 / trick
> 🔵 **Blue** = 后续要追溯的引用
> 🟣 **Purple** = 与我 thesis 直接相关
> ⚫ **Gray** = 背景 / 术语定义

## Annotations

%% begin annotations %%

### Imported 2026-05-22 10:36

- 🟡 **p.1** — a simulation-free approach for training CNFs based on regressing vector fields of fixed conditional probability paths. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=1&annotation=37YCMS4G)

### Imported 2026-05-24 14:19

- ⚫ **p.1** — CNF [⤴](zotero://open-pdf/library/items/Q69J7C42?page=1&annotation=S5BVL54H)
  - 💬 *我的批注*：cnf 可比 sde简单，$$\frac{dx_t}{dt}=v_t(x_t)$$，只不过sde的v通常易于解析并且已知，学习$/nabla log(p(x))$ ;而cnf v未知且复杂, 关于p(x)的知识全部含于v(x,t)中。

- 🟡 **p.1** — employing FM with diffusion paths results in a more robust and stable alternative for training diffusion models. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=1&annotation=WQZVIJ6G)

- 🔴 **p.1** — Furthermore [⤴](zotero://open-pdf/library/items/Q69J7C42?page=1&annotation=JKICIEZ5)
  - 💬 *我的批注*：FM 把问题从“如何设计一个 SDE”转成“如何设计一条好的 probability path，并学习其 velocity field”。

- 🟡 **p.1** — Optimal Transport (OT) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=1&annotation=NY6FA225)
  - 💬 *我的批注*：faster

- 🟡 **p.2** — a target vector field that generates a desired probability path [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=4DPUMB7B)

- 🟡 **p.2** — we can construct such target vector fields through per-example (i.e., conditional) formulations. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=L8NWPRUP)

- 🟡 **p.2** — we show that a per-example training objective, termed Conditional Flow Matching (CFM), provides equivalent gradients and does not require explicit knowledge of the intractable target vector field. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=XDTQ4BYH)
  - 💬 *我的批注*：wow

- 🟡 **p.2** — We find that conditional OT paths are simpler than diffusion paths, forming straight line trajectories whereas diffusion paths result in curved paths. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=TT6L6PMF)

- ⚫ **p.2** — diffeomorphic map [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=F6WKWC86)
  - 💬 *我的批注*：1. 它是可逆的； 2. 它和它的逆都是光滑的； 3. 不同初始点不会在有限时间内合并到同一个点。

- ⚫ **p.2** — modeling the vector field vt with a neural network, vt(x; θ),  where θ ∈ Rp are its learnable parameters, which in turn leads to a deep parametric model of the flow φt, called a Continuous Normalizing Flow (CNF). [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=7XU2CCJQ)

- 🟡 **p.2** — [φt]∗p0(x) = p0(φ−1  t (x)) det  [ ∂φ−1  t  ∂x (x)  ]  . (4) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=L8J7IYP9)
  - 💬 *我的批注*：原密度乘以体积压缩系数

- 🟡 **p.2** — continuity equation [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=9H6FR8FK)
  - 💬 *我的批注*：更方便的方法是检查 $p_t$ 和 $v_t$ 是否满足连续性方程：  
        
      $$  
      \frac{\partial p_t(x)}{\partial t}  
      +  
      \nabla \cdot \bigl(p_t(x)v_t(x)\bigr)  
      =  
      0.  
      $$

- 🟡 **p.2** — how to compute the probability p1(x) at an arbitrary point x ∈ Rd [⤴](zotero://open-pdf/library/items/Q69J7C42?page=2&annotation=QY3XUG27)
  - 💬 *我的批注*：$$  
      \log p_1(x_1)  
      =  
      \log p_0(x_0)  
      -  
      \int_0^1  
      \nabla \cdot v_t(x_t)\, dt.  
      $$  
      因为  
      $$  
      \frac{d}{dt}\log p_t(x_t)  
      =  
      -  
      \nabla \cdot v_t(x_t)  
      $$

### Imported 2026-05-24 16:02

- 🟡 **p.3** — There are many choices of probability paths that can satisfy p1(x) ≈ q(x) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=3&annotation=TWR9PXSQ)

- 🟡 **p.3** — In this section, we show that we can construct both pt and ut using probability paths and vector fields that are only defined per sample, and an appropriate method of aggregation provides the desired pt and ut [⤴](zotero://open-pdf/library/items/Q69J7C42?page=3&annotation=8QC3IPZB)
  - 💬 *我的批注*：$$  
      p_t(x)  
      =  
      \int  
      p_t(x\mid x_1)\, q(x_1)\, dx_1.  
      $$  
        
      $$  
      p_t(x)u_t(x)  
      =  
      \int  
      p_t(x\mid x_1)\,  
      u_t(x\mid x_1)\,  
      q(x_1)\,  
      dx_1.  
      $$  
        
      所以：  
        
      $$  
      u_t(x)  
      =  
      \frac{  
      \int  
      p_t(x\mid x_1)\,  
      u_t(x\mid x_1)\,  
      q(x_1)\,  
      dx_1  
      }{  
      p_t(x)  
      }.  
      $$

- 🟡 **p.3** — ut(x) =  ∫  ut(x|x1) pt(x|x1)q(x1)  pt(x) dx1, (8) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=3&annotation=2956S5BL)

- 🟡 **p.3** — The marginal vector field (equation 8) generates the marginal probability path (equation 6). [⤴](zotero://open-pdf/library/items/Q69J7C42?page=3&annotation=777DE26K)
  - 💬 *我的批注*：你把它想象为物理流（比如说气流水流试试）

- 🟡 **p.4** — L  CFM(θ) = Et,q(x1),pt(x|x1)  ∥ ∥vt(x) − ut(x|x1)∥  ∥  2, (9) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=RPQMBM29)

- 🟡 **p.4** — The FM (equation 5) and CFM (equation 9) objectives have identical gradients w.r.t. θ. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=I3ATXUQN)
  - 💬 *我的批注*：$$  
      \begin{aligned}  
      &  
      \mathbb{E}_{x_1,t,x}  
      \Big[  
      \big(  
      v_t(x)-u_t(x\mid x_1)  
      \big)^2  
      \Big]  
      \\  
      ={}&  
      \mathbb{E}_{x,t}  
      \left[  
      \mathbb{E}_{x_1\mid x,t}  
      \Big[  
      \big(  
      v_t(x)-u_t(x\mid x_1)  
      \big)^2  
      \Big]  
      \right]  
      \\  
      ={}&  
      \mathbb{E}_{x,t}  
      \left[  
      \mathbb{E}_{x_1\mid x,t}  
      \Big[  
      v_t(x)^2  
      -  
      2v_t(x)u_t(x\mid x_1)  
      +  
      u_t(x\mid x_1)^2  
      \Big]  
      \right]  
      \\  
      ={}&  
      \mathbb{E}_{x,t}  
      \Big[  
      v_t(x)^2  
      -  
      2v_t(x)  
      \mathbb{E}_{x_1\mid x,t}  
      \big[  
      u_t(x\mid x_1)  
      \big]  
      \\  
      &\qquad\qquad  
      +  
      \mathbb{E}_{x_1\mid x,t}  
      \big[  
      u_t(x\mid x_1)^2  
      \big]  
      \Big]  
      \\  
      ={}&  
      \mathbb{E}_{x,t}  
      \Big[  
      v_t(x)^2  
      -  
      2v_t(x)u_t(x)  
      +  
      \mathbb{E}_{x_1\mid x,t}  
      \big[  
      u_t(x\mid x_1)^2  
      \big]  
      \Big].  
      \end{aligned}  
      $$

- 🟡 **p.4** — Theorem 2. Assuming that pt(x) > 0 for all x ∈ Rd and t ∈ [0, 1], then, up to a constant independent of θ, LCFM and LFM are equal. Hence, ∇θLFM(θ) = ∇θLCFM(θ). [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=L788GCW2)

### Imported 2026-05-24 20:17

- 🟡 **p.4** — 4 CONDITIONAL PROBABILITY PATHS AND VECTOR FIELDS [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=ZW8SYAEP)
  - 💬 *我的批注*：一种实践派

- 🟡 **p.4** — pt(x|x1) = N (x | μt(x1), σt(x1)2I), (10) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=X8VXHTJE)

- 🟡 **p.4** — ψt(x) = σt(x1)x + μt(x1). (11) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=4XBFVAY2)
  - 💬 *我的批注*：起点是噪声

- 🟡 **p.4** — When x is distributed as a standard Gaussian [⤴](zotero://open-pdf/library/items/Q69J7C42?page=4&annotation=FF9K362N)

- 🟡 **p.5** — ut(x|x1) = σt′(x1)  σt(x1) (x − μt(x1)) + μ′  t(x1). (15) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=5&annotation=9PQZBXGT)
  - 💬 *我的批注*：显而易见

- 🟡 **p.5** — μt(x) = tx1, and σt(x) = 1 − (1 − σmin)t. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=5&annotation=KCNJFCXL)

- 🟡 **p.6** — x1 − (1 − σmin)x0 [⤴](zotero://open-pdf/library/items/Q69J7C42?page=6&annotation=9JUM3TK6)
  - 💬 *我的批注*：恒定方向和大小

- 🟡 **p.6** — The conditional flow ψt(x) is in fact the Optimal Transport (OT) displacement map between the two Gaussians p0(x|x1) and p1(x|x1). [⤴](zotero://open-pdf/library/items/Q69J7C42?page=6&annotation=ZIUINX3Q)

- 🟡 **p.6** — pt = [(1 − t)id + tψ]?p0 (24) [⤴](zotero://open-pdf/library/items/Q69J7C42?page=6&annotation=2LBMMZ63)

- 🟡 **p.6** — Figure 3: Diffusion and OT trajectories. [⤴](zotero://open-pdf/library/items/Q69J7C42?page=6&annotation=2AE9SLHD)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 还是接score的思路，我认为主要分概率图和sampling两个方面分析。概率图比较简单，由p(x)不断流到一个普世分布；sampling难一点，采样点加权的向各个数据点位置去走，按照该位置单数据点分布下在该时间的概率加权。
2. sampling和prob的关系如下$$\frac{\partial p_t(x)}{\partial t}+\nabla \cdot \bigl(p_t(x)u_t(x)\bigr)=0.$$p,u两者不能互推，但是特定限制情况下有互推方法
3. 前置知识：$$u_t(x)=\int u_t(x\mid x_1)\,\frac{p_t(x\mid x_1)\,q(x_1)}{p_t(x)}\,dx_1.$$，类比流体的$E(v)=\frac{E(\rho v)}{\rho}$.用连续性方程证明。
4. 当满足高斯条件路径$$p_t(x\mid x_1)=\mathcal{N}\bigl(x\mid\mu_t(x_1),\sigma_t(x_1)^2I\bigr).$$采样形式：$$x_t=\mu_t(x_1)+\sigma_t(x_1)x_0,\qquad
x_0\sim\mathcal{N}(0,I).$$对应条件速度：$$u_t(x\mid x_1)=\frac{\sigma_t'(x_1)}{\sigma_t(x_1)}\bigl(x-\mu_t(x_1)\bigr)+\mu_t'(x_1).$$然后就可以设计出$\mu_t(x_1)\sigma_t(x_1)$后互推了
5. 一种v设计：OT（关键实践优势）$$x_t=\bigl(1-(1-\sigma_{\min})t\bigr)x_0+tx_1.$$速度笔直不变。
6. 算法流程：
	1. 从数据集中采样，采样噪声，采样时间
	2. 用条件路径构造中间点$$x_t=\mu_t(x_1)+\sigma_t(x_1)x_0,\qquad
x_0\sim\mathcal{N}(0,I).$$
	3. 计算条件速度标签，训练网络
	4. 采样
7. 采样举例：初始点更靠近“dog”，那么“dog”的权重大，“cat”权重小，采样点逐渐向“dog”靠近。

%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/flow-matching]]、[[wiki/concepts/conditional-flow-matching]]、[[wiki/concepts/continuous-normalizing-flow]]、[[wiki/concepts/optimal-transport-path]]；关联 [[wiki/concepts/score-sde]]、[[wiki/concepts/probability-flow-ode]]、[[wiki/concepts/score-matching]]（CFM≡DSM）、[[wiki/concepts/fokker-planck-equation]]（连续性方程=无扩散退化）
- 方法：[[wiki/methods/ddpm]]（被收为高斯路径特例）、[[wiki/methods/ncsn]]、[[wiki/methods/ddim]]（DDIM=训diffusion+采flow；FM=连训练也flow化）、[[wiki/methods/rectified-flow]]（下游/并行）
- 实体（作者/机构）：[[wiki/entities/yaron-lipman]]、[[wiki/entities/ricky-chen]]、[[wiki/entities/meta-ai-fair]]
- 基线 / 对比：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/imagenet]]；同架构消融对比 DDPM / Score Matching / ScoreFlow（FM-OT 三项 NLL/FID/NFE 全胜）

%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
- ✅ 命中 overview 推论 1：FM 真换了训练目标（回归速度场而非 ε/score）却仍在"迭代生成+预测速度场+沿链注入条件"范式内——"训练目标可演化、范式不变"的最强样本。
- ✅ 强化推论 3：OT 直线路径 NFE 降到约 60%、训练期采样成本恒定 → 加速可来自"路径设计"而非只靠采样器/蒸馏。
- ⚠️ 给推论 2 加 caveat：FM 的 $t$ 是噪声→数据插值系数，与 diffusion 方向相反；做"介入时间步"分析前必须先统一坐标，否则结论会翻转。
- 已据此把 [[wiki/overview]] working thesis 升到 v0.4；待追：FM/RF 模型（SD3/FLUX）上的编辑方法怎么做 inversion。
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
- [ ]
- [ ]
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/lipmanFlowMatchingGenerative2023.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/lipmanFlowMatchingGenerative2023.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-24T20:17:16.220+08:00 %%
