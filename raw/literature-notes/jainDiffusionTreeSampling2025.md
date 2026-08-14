---
type: literature-note
citekey: jainDiffusionTreeSampling2025
title: "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"
aliases: ["@jainDiffusionTreeSampling2025"]
authors: "Vineet Jain, Kusha Sareen, Mohammad Pedramfar, Siamak Ravanbakhsh"
firstAuthor: "Jain"
year: 2025
itemType: preprint
doi: "10.48550/arXiv.2506.20701"
url: "http://arxiv.org/abs/2506.20701"
zotero: "zotero://select/library/items/TI986GR8"
tags: [literature, computer-science---artificial-intelligence, computer-science---machine-learning, statistics---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-14
updated: 2026-08-14
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/jainDiffusionTreeSampling2025]]" # e.g. "[[wiki/sources/jainDiffusionTreeSampling2025]]"
---

# Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models

> [!info] @jainDiffusionTreeSampling2025 · Jain et al. · 2025
> [Open in Zotero](zotero://select/library/items/TI986GR8) · [DOI](https://doi.org/10.48550/arXiv.2506.20701) · [URL](http://arxiv.org/abs/2506.20701) · [PDF](file:///home/lonper/Zotero/storage/I2FU9H89/Jain%20等%20-%202025%20-%20Diffusion%20Tree%20Sampling%20Scalable%20inference-time%20alignment%20of%20diffusion%20models.pdf)

## Abstract

> [!abstract]- Click to expand
> Adapting a pretrained diffusion model to new objectives at inference time remains an open problem in generative modeling. Existing steering methods suffer from inaccurate value estimation, especially at high noise levels, which biases guidance. Moreover, information from past runs is not reused to improve sample quality, resulting in inefficient use of compute. Inspired by the success of Monte Carlo Tree Search, we address these limitations by casting inference-time alignment as a search problem that reuses past computations. We introduce a tree-based approach that samples from the reward-aligned target density by propagating terminal rewards back through the diffusion chain and iteratively refining value estimates with each additional generation. Our proposed method, Diffusion Tree Sampling (DTS), produces asymptotically exact samples from the target distribution in the limit of infinite rollouts, and its greedy variant, Diffusion Tree Search (DTS⋆), performs a global search for high reward samples. On MNIST and CIFAR-10 class-conditional generation, DTS matches the FID of the best-performing baseline with up to 10× less compute. In text-to-image generation and language completion tasks, DTS⋆ effectively searches for high reward samples that match best-of-N with up to 5× less compute. By reusing information from previous generations, we get an anytime algorithm that turns additional compute into steadily better samples, providing a scalable approach for inference-time alignment of diffusion models1.

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
%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
这篇论文叫 **Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models**。它的核心想法可以压缩成一句话：

> **把 diffusion 的 inference-time guidance 看成一个 Monte Carlo Tree Search（MCTS）问题，并且让不同 generation 之间共享过去探索得到的 reward/value 信息。**

作者认为，现有 diffusion alignment 最大的问题不是“不会 guidance”，而是 **中间 noisy state 的 value 根本估不准**。DTS 就是专门针对这个问题设计的。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

### 1. 它到底要解决什么问题？

假设你已经有一个 pretrained diffusion：

[  
p_\theta(x)  
]

然后 inference 时突然给你一个 reward (r(x))，你希望生成的东西既符合原模型分布，又有高 reward。标准目标可以写成

# [  
\pi^*(x)

\frac1Z p_\theta(x)e^{\lambda r(x)}.  
]

也等价于一个 KL-regularized RL objective：

[  
\max_\pi  
\mathbb E_{x\sim\pi}[r(x)]  
-\frac1\lambda D_{\mathrm{KL}}(\pi|p_\theta).  
]

所以它本质上是：**不要重新训练 diffusion model，而是在 inference-time 从 reward-tilted posterior 中采样。** ([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

关键 quantity 是 soft value：

# [  
V_t(x_t)

\frac1\lambda  
\log  
\mathbb E_{p_\theta(x_{0:t-1}|x_t)}  
\left[e^{\lambda r(x_0)}\right].  
]

理想的 guided transition 恰好是

[  
\pi^*_t(x_{t-1}|x_t)  
\propto  
p_\theta(x_{t-1}|x_t)  
e^{\lambda V_{t-1}(x_{t-1})}.  
]

也就是说：

**只要你知道每个 noisy state 的 (V_t)，alignment 基本就解决了。** ([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

---

### 2. 作者认为现有方法卡在哪里？

问题就在于：**(V_t(x_t)) 很难算。**

很多 DPS、SMC、reward-guidance 方法实际上会做类似这样的近似：

[  
V_t(x_t)  
\approx  
\mathbb E[r(x_0)|x_t]  
\approx  
r(\hat x_0(x_t)),  
]

其中 (\hat x_0(x_t)) 通常通过 Tweedie formula 从 (x_t) 一步预测 clean sample。

但是 (t) 很大、noise 很强的时候，(\hat x_0) 本身就非常不靠谱。论文的 2D toy experiment 显示，到了 high-noise region，这种 one-step prediction 甚至接近随机，因此 guidance 用的 value 也是严重 biased 的。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

而 Best-of-N、SMC 等还有另一个问题：

**每次 generation 基本是独立的。**

例如你已经 rollout 了 100 条 trajectory，知道某些 early noisy states 最终往往会走向高 reward 区域；下一次生成时，这些信息通常全部丢掉。

所以作者提出：

> 与其每次重新从头采样，为什么不把所有 diffusion trajectories 存成一棵树？

---

## 3. DTS：把 diffusion trajectory 变成搜索树

这是整篇论文最重要的 idea。

reverse diffusion：

[  
x_T\rightarrow x_{T-1}\rightarrow\dots\rightarrow x_0  
]

被看成一棵树。

一个 (x_t) 是一个 **node**，从

[  
p_\theta(x_{t-1}|x_t)  
]

采不同 noise realization，就得到不同 child。

于是变成：

```text
                    x_T
                /    |    \
           x_{T-1} x_{T-1} x_{T-1}
             /  \       ...
          ...
            |
           x_0
          reward
```

每个 node 保存：

[  
(x_t,\ t,\ \hat V_t(x_t),\ N(x_t)).  
]

然后像经典 MCTS 一样不断做四件事：([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

1. **Selection**：根据已有的 value 选择 promising branch：
    

[  
P(x_{t-1}|x_t)  
\propto e^{\lambda\hat V_{t-1}(x_{t-1})}.  
]

2. **Expansion**：从 diffusion transition
    

[  
x_{t-1}\sim p_\theta(\cdot|x_t)  
]

产生新的 child。

3. **Rollout**：从这个节点正常跑 diffusion，一直跑到 (x_0)。
    
4. **Backup**：计算 terminal reward
    

[  
V_0(x_0)=r(x_0),  
]

然后沿 trajectory **往回更新所有 ancestor 的 soft value**。

所以最关键的机制就是：

[  
\boxed{\text{terminal reward}  
\rightarrow  
\text{backpropagate through the diffusion tree}}  
]

这不是 gradient backprop，而是 **MCTS / soft Bellman backup**。

多跑几次之后，同一个 high-noise node 的 value 就不是依赖一次非常不准确的 (\hat x_0)，而是来自很多实际 rollout 的 Monte Carlo estimate。

因此：

[  
\text{more inference compute}  
\Rightarrow  
\text{better value estimation}  
\Rightarrow  
\text{better future sampling}.  
]

这就是作者所谓的 **anytime inference algorithm**。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

---

## 4. DTS 和 DTS(^\star) 是两个不同目标

这个区别很重要。

**DTS** 的目标是真正从

[  
\pi^*(x)\propto p_\theta(x)e^{\lambda r(x)}  
]

里 **sampling**。

selection 是 Boltzmann sampling：

[  
P(\text{child})  
\propto  
e^{\lambda\hat V}.  
]

论文还证明，在 bounded reward 等条件下，随着 tree rollout 数

[  
M\rightarrow\infty,  
]

terminal samples 的 empirical distribution 会收敛到目标 (\pi^*)。换句话说，理论 claim 不只是“找到高 reward sample”，而是 **asymptotically correct posterior sampling**。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

而 **DTS(^\star)** 更像真正的 MCTS search，它想解决：

[  
\text{find one very good }x.  
]

因此 selection 变成带 exploration bonus 的 UCT：

# [  
\operatorname{UCT}(x)

\hat V(x)  
+  
c_{\mathrm{uct}}  
\sqrt{  
\frac{\log N(\text{parent})}{N(x)}  
}.  
]

它适合 text-to-image、文本生成这种“我就想找最好的几个 sample”的任务。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

一个很漂亮的地方是，作者没有简单地 maximize terminal reward，而是利用 **soft value / posterior mass**。

所以某个 extremely high-reward、但在 pretrained model 下 probability 极小的 pathological sample，不一定会被选中。

这也是他们解释 DTS(^\star) 为什么比某些 SMC 方法更不容易 reward hacking / over-optimization 的原因。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

---

## 5. 实验结果在说明什么？

他们做了几个层次。

在 2D Gaussian mixture / checkerboard toy problem 中，DTS 更准确地恢复

[  
p(x)e^{r(x)}  
]

的真实 target distribution；同时随着 NFEs 增加，MMD 持续下降。更关键的是，他们直接测了 value estimator 的 bias/variance：Tweedie one-step estimate 和 single rollout 在 high noise 时都有明显 bias/variance，而 tree aggregation 显著改善。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

在 MNIST / CIFAR-10 class-conditional sampling 中，DTS 在 FID、CMMD 上整体优于 DPS、SMC/FK、TDS、DAS。例如论文报告的 (10^6) NFEs 下，DTS 在 CIFAR-10 上 FID 为 (0.195)，对比 DAS 的 (0.241)、SMC/FK 的 (0.313)；作者还观察到一些 SMC-based 方法出现 mode collapse，而 DPS 会产生偏离 prior support 的样本。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

在 Stable Diffusion v1.5 + ImageReward / aesthetic reward 上，DTS(^\star) 随着 inference compute 增加的 scaling 明显优于 Best-of-N；SMC 有时 reward 数字更高，但视觉上出现明显 reward over-optimization，而 DTS(^\star) 更能保持 base model fidelity。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

他们还在 discrete diffusion LM（MDLM）上做文本生成，DTS(^\star) 随 compute 增加持续获得更高 reward，同时保持更高 trigram diversity，而 SMC 更容易为 reward 牺牲 diversity。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

论文摘要概括的 compute claim 是：class-conditional generation 上可用最高约 **10× 更少 compute** 匹配最强 baseline，text-to-image / language completion 上约 **5×**。([arXiv](https://arxiv.org/abs/2506.20701 "[2506.20701] Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

---

### 我觉得这篇论文最值得抓住的不是“MCTS + diffusion”

真正有意思的是这个视角：

[  
\boxed{  
\text{Inference-time scaling}  
\neq  
\text{independent Best-of-N}  
}  
]

而应该是

[  
\boxed{  
\text{additional compute}  
\rightarrow  
\text{accumulate knowledge about the generative trajectory}  
}  
]

Best-of-N：

# [  
1000\text{ samples}

1000\text{ almost independent trajectories}.  
]

DTS：

# [  
1000\text{ rollouts}

\text{one increasingly well-estimated value landscape}.  
]

所以它实际上是在问一个更 general 的问题：

> **生成模型 inference-time compute 能不能像 AlphaZero search 一样，后面的计算利用前面的计算结果？**

这点我认为比具体 algorithm 更重要。

对你做 diffusion / flow 来说，还有一个很自然的研究方向：**DTS 本身高度依赖 stochastic reverse transition 产生 branching；如果换到 deterministic probability-flow ODE / flow matching，应该怎样构造可复用的 trajectory tree？** 论文自己指出该 tree framework原则上作用于 Markov chain，而高维 search 能成功很依赖 pretrained generative model 作为强 prior；因此怎么把这种 “value backup + reusable inference compute” 搬到 flow/ODE setting，是一个很自然但并非这篇论文已经解决的问题。([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))

如果只记住这篇文章的 **三个公式**，就是

[  
\pi^*(x)\propto p_\theta(x)e^{\lambda r(x)},  
]

# [  
V_t(x_t)

\frac1\lambda  
\log  
\mathbb E[e^{\lambda r(x_0)}|x_t],  
]

以及

[  
\pi_t^*(x_{t-1}|x_t)  
\propto  
p_\theta(x_{t-1}|x_t)  
e^{\lambda V_{t-1}(x_{t-1})}.  
]

**DTS 所做的一切，本质上就是用 tree rollout + soft Bellman backup，把第三个式子里的 (V) 越估越准。** ([arXiv](https://arxiv.org/html/2506.20701v1 "Diffusion Tree Sampling: Scalable inference-time alignment of diffusion models"))
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
> `ingest raw/literature-notes/jainDiffusionTreeSampling2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/jainDiffusionTreeSampling2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-14T15:24:14.735+08:00 %%
