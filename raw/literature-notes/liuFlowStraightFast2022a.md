---
type: literature-note
citekey: liuFlowStraightFast2022a
title: "Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow"
aliases:
  - "@liuFlowStraightFast2022a"
authors: Xingchao Liu, Chengyue Gong, Qiang Liu
firstAuthor: Liu
year: 2022
itemType: preprint
doi: 10.48550/arXiv.2209.03003
url: http://arxiv.org/abs/2209.03003
zotero: zotero://select/library/items/W34L9UAV
tags:
  - literature
  - todo
status: unread
priority: P2
my-rating: "5"
created: 2026-05-26
updated: 2026-05-26
ingested_to_wiki: true
wiki_page: "[[wiki/sources/liuFlowStraightFast2022a]]"
---

# Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow

> [!info] @liuFlowStraightFast2022a · Liu et al. · 2022
> [Open in Zotero](zotero://select/library/items/W34L9UAV) · [DOI](https://doi.org/10.48550/arXiv.2209.03003) · [URL](http://arxiv.org/abs/2209.03003) · [PDF](file:///home/lonper/Zotero/storage/M4N9AKDH/Liu%20等%20-%202022%20-%20Flow%20Straight%20and%20Fast%20Learning%20to%20Generate%20and%20Transfer%20Data%20with%20Rectified%20Flow.pdf)

## Abstract

> [!abstract]- Click to expand
> We present rectified flow, a surprisingly simple approach to learning (neural) ordinary differential equation (ODE) models to transport between two empirically observed distributions π_0 and π_1, hence providing a unified solution to generative modeling and domain transfer, among various other tasks involving distribution transport. The idea of rectified flow is to learn the ODE to follow the straight paths connecting the points drawn from π_0 and π_1 as much as possible. This is achieved by solving a straightforward nonlinear least squares optimization problem, which can be easily scaled to large models without introducing extra parameters beyond standard supervised learning. The straight paths are special and preferred because they are the shortest paths between two points, and can be simulated exactly without time discretization and hence yield computationally efficient models. We show that the procedure of learning a rectified flow from data, called rectification, turns an arbitrary coupling of π_0 and π_1 to a new deterministic coupling with provably non-increasing convex transport costs. In addition, recursively applying rectification allows us to obtain a sequence of flows with increasingly straight paths, which can be simulated accurately with coarse time discretization in the inference phase. In empirical studies, we show that rectified flow performs superbly on image generation, image-to-image translation, and domain adaptation. In particular, on image generation and translation, our method yields nearly straight flows that give high quality results even with a single Euler discretization step.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 加深一下印象
- 学习recflow方法
- 学习一下概率语言，去decorate一下论文，以实现升咖。
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

### Imported 2026-05-26 09:32

- 🟡 **p.1** — We show that the procedure of learning a rectified flow from data, called rectification, turns an arbitrary coupling of π0 and π1 to a new deterministic coupling with provably non-increasing convex transport costs. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=1&annotation=SX83VT7M)

- 🟡 **p.2** — The Transport Mapping Problem Given empirical observations of two distributions X0 ∼ π0, X1 ∼ π1 on Rd, find a transport map T : Rd → Rd (hopefully nice or optimal in certain sense), such that Z1 := T (Z0) ∼ π1 when Z0 ∼ π0, that is, (Z0, Z1) is a coupling (a.k.a transport plan) of π0 and π1. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=2&annotation=6XXRCXQX)
  - 💬 *我的批注*：主要问题

- 🟡 **p.3** — The rectified flow is an ODE model that transport distribution π0 to π1 by following straight line paths as much as possible. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=3&annotation=FMRYSZKW)

- 🟡 **p.3** — Algorithmically, the rectified flow is trained with a simple and scalable unconstrained least squares optimization procedure, which avoids the instability issues of GANs, the intractable likelihood of MLE methods, and the subtle hyper-parameter decisions of denoising diffusion models. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=3&annotation=BEF77JD5)

- 🟡 **p.5** — Flows avoid crossing [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=5&annotation=YN42ELV6)
  - 💬 *我的批注*：auto,不信你画个图

- 🟡 **p.5** — Rectified flows reduce transport costs [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=5&annotation=NAE5T4MG)

### Imported 2026-05-26 11:41

- 🟡 **p.5** — Straight line flows yield fast simulation [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=5&annotation=MNNQBZI6)

- 🟡 **p.6** — Marginal preserving property [Theorem 3.3] The pair (Z0, Z1) is a coupling of π0 and π1. In fact, the marginal law of Zt equals that of Xt at every time t, that is, Law(Zt) = Law(Xt), ∀t ∈ [0, 1]. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=6&annotation=GBSUPUH9)

- 🟡 **p.6** — Reducing transport costs [Theorem 3.5] The coupling (Z0, Z1) yields lower or equal convex transport costs than the input (X0, X1) in that E[c(Z1 − Z0)] ≤ E[c(X1 − X0)] for any convex cost c : Rd → R. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=6&annotation=CQJXGYAK)

- 🟡 **p.7** — Reflow, straightening, fast simulation [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=7&annotation=3U2DX3AL)

- 🟡 **p.8** — Distillation [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=E9LBQSDR)

- 🟡 **p.8** — On the velocity field vX [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=7YYPSZ8T)

- 🟡 **p.8** — vX (z, t) = E  [ X1 − z  1 − t ηt(X1, z)  ]  , ηt(X1, z) = ρ  ( z − tX1  1−t  ∣ ∣ ∣ ∣  X1  )/  E  [  ρ  ( z − tX1  1−t  ∣ ∣ ∣ ∣  X1  )]  , (4) [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=53DKCNUE)

- 🟡 **p.8** — A simple fix is to add X0 with a Gaussian noise ξ ∼ N (0, σ2I) independent of (X0, X1) to yield a smoothed variable X ̃0 = X0 + ξ, and transfer X ̃0 to X1 using rectified flow. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=EKI5LP5L)
  - 💬 *我的批注*：smoother,为了满足$\rho(x_0|x_1)$

- 🟡 **p.8** — Smooth function approximation [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=Z7EES2LG)

- 🔴 **p.8** — This, however, is not practically useful in most cases as it completely overfits the  8 data. [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=8&annotation=9WAV7JTT)
  - 💬 *我的批注*：同flowcycle

- 🟡 **p.9** — vX,h(z, t) = E  [ X1 − z  1 − t ωh(Xt, z)  ]  , (5) [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=9&annotation=T8PFPZVE)
  - 💬 *我的批注*：for toy model

- 🟡 **p.9** — A Nonlinear Extension [⤴](zotero://open-pdf/library/items/M4N9AKDH?page=9&annotation=XJFPGDKB)
  - 💬 *我的批注*：就是上一篇FLOW MATCHING FOR GENERATIVE MODELING的方法

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
%% begin my-summary %%

1. Rectified Flow 的核心算法是：给定 coupling $(X_0,X_1)$，构造直线插值  
    $$  
    X_t=(1-t)X_0+tX_1  
    $$  
    并训练速度场  
    $$  
    v_\theta(X_t,t)\approx X_1-X_0.  
    $$  
    训练目标本质是普通 MSE 回归：  
    $$  
    \mathbb E_{t,(X_0,X_1)}  
    \left[  
    |v_\theta(X_t,t)-(X_1-X_0)|^2  
    \right].  
    $$
    平方损失下的理想解是边缘速度场：  
    $$
    v^\ast(x,t)=\mathbb E[X_1-X_0\mid X_t=x].  
    $$
    
2. Rectified Flow 把非因果的线性插值过程 $X_t$ 变成因果 ODE：  
    $$ 
    dZ_t=v_\theta(Z_t,t),dt.  
    $$  
    线性插值路径可以交叉，因为它们依赖完整端点对 $(X_0,X_1)$；但 ODE flow 在同一 $(x,t)$ 处只能有一个速度，所以轨迹不能交叉。于是 Rectified Flow 会在交叉/速度冲突处 rewiring，改变原始 coupling，形成新的 deterministic coupling：  
    $$ 
    Z_1=\Phi_1(Z_0).  
    $$  
    它改变的是整个 coupling，而不仅是局部交叉点。(当然最直观的就是交叉点如下图所示)![[liuFlowStraightFast2022a-1779768003417.webp]]
    
3. sampling 时，从  
    $$  
    Z_0\sim\pi_0  
    $$  
    出发解 ODE：  
    $$ 
    \frac{dZ_t}{dt}=v_\theta(Z_t,t),\qquad t:0\to1.  
    $$  
    理想情况下，由于  
    $$ 
    v^\ast(x,t)=\mathbb E[\dot X_t\mid X_t=x]  
    $$  
    生成与 $X_t$ 相同的边缘分布路径，所以  
    $$  
    \operatorname{Law}(Z_t)=\operatorname{Law}(X_t),  
    $$  
    特别地：  
    $$ 
    Z_1\sim\pi_1.  
    $$  
    实际采样可用 Euler：  
    $$ 
    Z_{t+h}=Z_t+h,v_\theta(Z_t,t).  
    $$
    
4. reflow 是反复用上一轮 ODE 诱导的新 coupling 再训练 Rectified Flow：  
    $$
    Z^{k+1}=\operatorname{RectFlow}((Z_0^k,Z_1^k)).  
    $$  
    它会降低所有凸传输代价：  
    $$ 
    \mathbb E[c(Z_1-Z_0)]\le \mathbb E[c(X_1-X_0)]  
    $$  
    并使路径越来越 straight。straightness 可用  
    $$ 
    S(Z)=\int_0^1\mathbb E\left[|(Z_1-Z_0)-\dot Z_t|^2\right]dt  
    $$  
    衡量；$S(Z)=0$ 表示常速直线。
    
5. 直线路径带来快速 sampling：如果 flow 完全 straight，则  
    $$
    Z_t=(1-t)Z_0+tZ_1,\qquad v(Z_t,t)=Z_1-Z_0,  
    $$  
    单步 Euler 就精确：  
    $$ 
    Z_1=Z_0+v(Z_0,0).  
    $$  
    因此 Rectified Flow 的目标不是一般高维全局 OT，而是通过 rectification/reflow 得到更 deterministic、更低凸代价、更 straight、可 few-step/one-step 采样的 coupling。 
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
-
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/liuFlowStraightFast2022a.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/liuFlowStraightFast2022a.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-26T11:41:57.725+08:00 %%
