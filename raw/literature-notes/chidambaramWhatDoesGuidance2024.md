---
type: literature-note
citekey: chidambaramWhatDoesGuidance2024
title: "What does guidance do? A fine-grained analysis in a simple setting"
aliases: ["@chidambaramWhatDoesGuidance2024"]
authors: "Muthu Chidambaram, Khashayar Gatmiry, Sitan Chen, Holden Lee, Jianfeng Lu"
firstAuthor: "Chidambaram"
year: 2024
itemType: preprint
doi: "10.48550/arXiv.2409.13074"
url: "http://arxiv.org/abs/2409.13074"
zotero: "zotero://select/library/items/CFIMGHRS"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---machine-learning, statistics---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-14
updated: 2026-08-14
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/chidambaramWhatDoesGuidance2024]]"
---

# What does guidance do? A fine-grained analysis in a simple setting

> [!info] @chidambaramWhatDoesGuidance2024 · Chidambaram et al. · 2024
> [Open in Zotero](zotero://select/library/items/CFIMGHRS) · [DOI](https://doi.org/10.48550/arXiv.2409.13074) · [URL](http://arxiv.org/abs/2409.13074) · [PDF](file:///home/lonper/Zotero/storage/SXR47RYD/Chidambaram%20等%20-%202024%20-%20What%20does%20guidance%20do%20A%20fine-grained%20analysis%20in%20a%20simple%20setting.pdf)

## Abstract

> [!abstract]- Click to expand
> The use of guidance in diffusion models was originally motivated by the premise that the guidance-modified score is that of the data distribution tilted by a conditional likelihood raised to some power. In this work we clarify this misconception by rigorously proving that guidance fails to sample from the intended tilted distribution. Our main result is to give a fine-grained characterization of the dynamics of guidance in two cases, (1) mixtures of compactly supported distributions and (2) mixtures of Gaussians, which reflect salient properties of guidance that manifest on real-world data. In both cases, we prove that as the guidance parameter increases, the guided model samples more heavily from the boundary of the support of the conditional distribution. We also prove that for any nonzero level of score estimation error, sufficiently large guidance will result in sampling away from the support, theoretically justifying the empirical finding that large guidance results in distorted generations. In addition to verifying these results empirically in synthetic settings, we also show how our theoretical insights can offer useful prescriptions for practical deployment.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
-
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

### Imported 2026-08-14 10:54

- 🟡 **p.1** — the guided model samples more heavily from the boundary of the support of the conditional distribution. [⤴](zotero://open-pdf/library/items/SXR47RYD?page=1&annotation=7ZIF48ID)

- 🟡 **p.4** — analysis for mixtures of compactly supported distributions:  Theorem 1 (Compactly supported setting, informal – see Theorem 4). Consider a data distribution p=1  2p (1) + 1  2p (−1) where p (1), p (−1) are β-bounded and supported on disjoint intervals [α1, α2] and  [−α2, −α1] respectively (see Assumption 1). Suppose that one runs the probability flow ODE with guidance  parameter w which is larger than some absolute constant. Then with probability 1 − e −Ω(w) , the resulting sample lies in the interval  α2 1 − O (1/  √  ln w ) , α2 ,  where the O (·) notation hides constants depending on α1, α2, β. [⤴](zotero://open-pdf/library/items/SXR47RYD?page=4&annotation=PHCQ9MD8)
  - 💬 *我的批注*：可以把整篇理论部分记成下面这条链： Theorem 1紧支撑：large guidance→目标类最远边界⇓Theorem 2Gaussian：没有边界→跑进 tail⇓Theorem 3真实模型 tail score 有误差+large guidance→可能 off-support / distorted generation​​换成人话： <b>CFG 越大，越倾向于生成目标类别中特别“典型”、特别远离其他类别的样本；但它也因此越来越容易把轨迹推入训练数据罕见的尾部区域，而这些地方的 score 最不可靠，于是 guidance 再继续增大就可能开始破坏生成质量。</b>

- 🟡 **p.5** — Theorem 2 (Gaussian setting). Consider the data distribution p = 1  2 N (1, 1) + 1  2 N (−1, 1). Suppose that one runs the probability flow ODE with guidance parameter w which is larger than some absolute constant. Then if the resulting sample is denoted by x ̃(1), we have  P(x ̃(1) ≥ 0) ≥ 1 − e −Ω(w2) P(x ̃(1) ≥ √  w) ≥ 1 − e−Ω(w) . [⤴](zotero://open-pdf/library/items/SXR47RYD?page=5&annotation=PHQX54WC)

- 🟡 **p.5** — Degradation when guidance is too large [⤴](zotero://open-pdf/library/items/SXR47RYD?page=5&annotation=M77HUCYD)

- 🟡 **p.5** — Theorem 3 [⤴](zotero://open-pdf/library/items/SXR47RYD?page=5&annotation=MR45LVZX)

- 🟡 **p.5** — then with probability at least 1 − e −Ω(w) , the resulting sample lies outside of the domain of p. [⤴](zotero://open-pdf/library/items/SXR47RYD?page=5&annotation=JRFV6B2J)

- 🟡 **p.5** — we should choose the guidance strength as large as possible while still ensuring that final samples are contained within the distribution support. [⤴](zotero://open-pdf/library/items/SXR47RYD?page=5&annotation=G4VK7B4L)

- 🟡 **p.7** — x′ (t) = x (t) + ∇ log pt (x (t)|z = 1) , (4) [⤴](zotero://open-pdf/library/items/SXR47RYD?page=7&annotation=G337H52Q)

- 🟡 **p.8** — so it is not a priori clear what the distribution over the final iterate x (T ) actually is. [⤴](zotero://open-pdf/library/items/SXR47RYD?page=8&annotation=JWQKQNYU)

- 🟡 **p.8** — 2.2 Intuition for our characterization of the dynamics of guidance [⤴](zotero://open-pdf/library/items/SXR47RYD?page=8&annotation=XM4SMXU6)
  - 💬 *我的批注*：所以作者真正想让你形成的图像是越像错误类⇒guidance 越强​而越像正确类⇒guidance 的额外作用越弱.​这会产生一个非对称效果：如果目标类在右边、错误类在左边，那么目标类内部<b>越靠右、越远离错误类的点，受到“被推走”的作用越小</b>；反过来，靠近错误类的一侧会受到更强的推力。于是最终就形成样本偏向目标类别中远离另一类别的区域.​这就是后面所谓的 archetype / extreme-point concentration 的直觉来源。

- 🟡 **p.8** — (w + 1)∇ log pt (x (t)|z) − w ∇ log pt (x (t)) ≈ (2w + 1) log pt (x (t)|z), [⤴](zotero://open-pdf/library/items/SXR47RYD?page=8&annotation=MIBVV48Q)
  - 💬 *我的批注*：(w+1)∇logpt​(x(t)∣z)−w∇logpt​(x(t))≈(2w+1)∇logpt​(x(t)∣z)

- 🟡 **p.27** — 6 Experiments [⤴](zotero://open-pdf/library/items/SXR47RYD?page=27&annotation=WJAWB6G4)
  - 💬 *我的批注*：所以整个 Section 6 可以压缩成一条非常清楚的故事： Toy data⇓真的看到 overshoot + pullback⇓MNIST⇓类似现象仍然存在，monotonicity 可用于选 w⇓ImageNet⇓简单几何不再成立，但 large guidance 与 off-support/error/distortion 仍有联系​​如果只记实验部分的结论，我建议记两句：对近似可分的情形：选“轨迹还基本单调”时最大的 guidance。​以及对复杂真实数据：尽量增大 guidance，但别增大到样本开始出现明显 off-support / distortion。​所以实验部分本质上是在给前面的理论加一句实践解释：<b>guidance 的 sweet spot 可能对应于“刚好还没进入危险 overshoot / off-support regime”的最大 guidance。</b>

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
这篇 NeurIPS 2024 论文可以压成一句话：

> **CFG 并不是在简单地采样一个 tilted distribution；它会在 reverse dynamics 中产生一种几何偏置，把样本推向目标类别中远离其他类别的极端区域，而过大的 guidance 还会放大尾部 score error，最终造成失真甚至 off-support。**

整篇文章的逻辑很清楚。

作者先质疑 CFG 的经典解释。通常我们写

[  
p^{z,w}(x)\propto p(x),p(z\mid x)^{1+w},  
]

并注意到在 (t=0) 时

# [  
\nabla\log p^{z,w}

(1+w)\nabla\log p(x\mid z)-w\nabla\log p(x).  
]

这看起来正好就是 CFG 的形式，所以人们容易认为：CFG 就是在采样这个 tilted distribution。

但 diffusion 真正需要的是每个噪声时刻 (t) 的 score。真正的目标应该是“先 tilt，再加噪”所得分布的

[  
\nabla\log p_t^{z,w}.  
]

而 CFG 用的是

[  
(1+w)\nabla\log p_t(x\mid z)-w\nabla\log p_t(x),  
]

也就是相当于“先加噪，再 tilt”。

关键就在：

[  
\boxed{\text{noising 和 tilting 一般不交换}}  
]

所以

[  
\boxed{  
\nabla\log p_t^{z,w}  
\neq  
(1+w)\nabla\log p_t(x\mid z)-w\nabla\log p_t(x).  
}  
]

因此 CFG 并不是严格地从原先想象的 tilted distribution 中采样。

接下来论文真正想回答的问题是：**那 CFG 实际在做什么？**

作者从 guided ODE 的动力学出发，得到一个核心直觉：如果当前粒子看起来更像错误类别，那么 guidance 的纠正作用很强；如果粒子已经明显进入正确类别，guidance 的额外作用就显著变弱。

所以大致有

[  
\text{错误类附近：强推}  
\rightarrow  
\text{中间区域：仍然强推}  
\rightarrow  
\text{正确类内部：推力减弱}.  
]

这会产生一种非常具体的几何偏置：

[  
\boxed{  
\text{guidance 偏爱目标类别中距离其他类别更远的位置。}  
}  
]

这也是文章最重要的机制性解释。

作者随后用三个定理把这套直觉严格化。

- **Theorem 1：紧支撑分布。** 假设正类支撑在 ([\alpha_1,\alpha_2])，负类支撑在 ([-\alpha_2,-\alpha_1])。当 guidance (w) 足够大时，最终样本会高概率集中到  
    [  
    x\approx\alpha_2,  
    ]  
    也就是正类中离负类最远的边界。因此 guidance 会造成 diversity collapse，并生成越来越“极端”“原型化”的样本。
    
- **Theorem 2：高斯混合。** 如果目标类是 (\mathcal N(1,1))，另一类是 (\mathcal N(-1,1))，由于高斯没有有限支撑边界，guidance 就会不断把样本推向目标高斯的尾部。作者证明典型样本会至少达到  
    [  
    x\gtrsim \sqrt w.  
    ]  
    所以紧支撑情形的“跑到边界”和高斯情形的“跑进 tail”，其实是同一种几何机制。
    
- **Theorem 3：score estimation error。** 即使 score 的整体 (L^2) 误差非常小，也不代表 large guidance 下采样一定稳定。原因是 guidance 会把轨迹推到低密度 tail，而 (L^2(p_t)) 对这些区域的误差控制很弱。于是很小的 tail score error 可能被 large guidance 放大，使轨迹无法返回数据 support，最终产生失真或 off-support 样本。
    

第三节、第四节的大量技术推导，本质上都只是在严格证明上面这些动力学图景。特别是在紧支撑情形里，粒子并不是平稳走到边界，而是会出现

[  
\text{强力向目标类运动}  
\rightarrow  
\text{冲过 support}  
\rightarrow  
\text{进入 tail}  
\rightarrow  
\text{再返回}  
\rightarrow  
\text{最后卡在边界附近}.  
]

这个 overshoot–return 结构也成为作者解释“大 guidance 为什么容易坏”的关键。

实验部分主要是理论的定性验证，而不是强实证结论。toy experiment 明确看到 overshoot 和 pullback；MNIST 上也能看到类似 trajectory non-monotonicity；ImageNet 上则只能看到一些与理论一致的现象，例如 guidance 增大后失真和某些 off-support proxy 增加。ImageNet 部分并不在理论定理的适用范围内，所以证据相对弱。

作者最后提出一个实践上的 heuristic：

[  
\boxed{  
\text{把 guidance 开到尽可能大，但不要大到轨迹开始明显 overshoot / off-support。}  
}  
]

因此整篇论文最值得记住的，其实不是单独某个定理，而是下面这条完整逻辑：

[  
\boxed{  
\text{CFG 不是静态 density reweighting}  
}  
]

而是

[  
\boxed{  
\text{一个随时间变化的几何 transport bias}.  
}  
]

它解释了为什么

[  
\text{guidance}\uparrow  
]

通常会同时带来

[  
\text{conditional fidelity}\uparrow,\qquad  
\text{diversity}\downarrow,  
]

而当 guidance 继续增大时，又可能出现

[  
\text{quality}\downarrow.  
]

如果只记最终结论，可以记这三句：

[  
\boxed{\text{1. CFG generally does not sample the naïve tilted distribution.}}  
]

[  
\boxed{\text{2. Guidance pushes samples toward target-class regions far from competing classes.}}  
]

[  
\boxed{\text{3. Large guidance exposes and amplifies tail score errors, producing a sweet spot.}}  
]

这三句话基本就是整篇文章。
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
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/chidambaramWhatDoesGuidance2024.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/chidambaramWhatDoesGuidance2024.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-14T10:54:33.041+08:00 %%
