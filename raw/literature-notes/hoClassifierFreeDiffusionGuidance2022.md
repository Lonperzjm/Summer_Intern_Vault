---
type: literature-note
citekey: hoClassifierFreeDiffusionGuidance2022
title: "Classifier-Free Diffusion Guidance"
aliases: ["@hoClassifierFreeDiffusionGuidance2022"]
authors: "Jonathan Ho, Tim Salimans"
firstAuthor: "Ho"
year: 2022
itemType: preprint
doi: "10.48550/arXiv.2207.12598"
url: "http://arxiv.org/abs/2207.12598"
zotero: "zotero://select/library/items/46P9X3ET"
tags: [literature, computer-science---artificial-intelligence, computer-science---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-11
updated: 2026-08-11
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/hoClassifierFreeDiffusionGuidance2022]]"
---

# Classifier-Free Diffusion Guidance

> [!info] @hoClassifierFreeDiffusionGuidance2022 · Ho et al. · 2022
> [Open in Zotero](zotero://select/library/items/46P9X3ET) · [DOI](https://doi.org/10.48550/arXiv.2207.12598) · [URL](http://arxiv.org/abs/2207.12598) · [PDF](file:///home/lonper/Zotero/storage/ZSSPLIMP/Ho和Salimans%20-%202022%20-%20Classifier-Free%20Diffusion%20Guidance.pdf)

## Abstract

> [!abstract]- Click to expand
> Classifier guidance is a recently introduced method to trade off mode coverage and sample fidelity in conditional diffusion models post training, in the same spirit as low temperature sampling or truncation in other types of generative models. Classifier guidance combines the score estimate of a diffusion model with the gradient of an image classifier and thereby requires training an image classifier separate from the diffusion model. It also raises the question of whether guidance can be performed without a classifier. We show that guidance can be indeed performed by a pure generative model without such a classifier: in what we call classifier-free guidance, we jointly train a conditional and an unconditional diffusion model, and we combine the resulting conditional and unconditional score estimates to attain a trade-off between sample quality and diversity similar to that obtained using classifier guidance.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 看看cfg
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
这篇就是 **Classifier-Free Diffusion Guidance（CFG）** 的原始论文，Jonathan Ho 和 Tim Salimans 写的。它后来几乎成了条件 diffusion / text-to-image 的标准组件。论文的核心问题很简单：

> **能不能不额外训练一个 classifier，也让 diffusion model 在采样时更“听条件的话”？**

答案就是今天大家熟悉的 **Classifier-Free Guidance**。([arXiv](https://arxiv.org/abs/2207.12598 "[2207.12598] Classifier-Free Diffusion Guidance"))

---

## 1. 它到底想解决什么问题？

先假设我们有一个 class-conditional diffusion：

[  
p_\theta(x\mid c),  
]

比如给定条件 (c=\text{dog})，生成狗的图片。

普通 conditional diffusion 虽然知道 (c)，但生成结果未必非常符合条件；有时提高条件一致性、单样本质量，需要某种 **guidance**。

在 CFG 之前，Dhariwal & Nichol 的 **classifier guidance** 做法是额外训练一个 noisy classifier：

[  
p_\phi(c\mid x_t),  
]

然后在 reverse diffusion 中加入 classifier 的梯度

[  
\nabla_{x_t}\log p_\phi(c\mid x_t).  
]

直觉上，它告诉 diffusion：

> “往 classifier 更确信这是 (c) 的方向走。”

这样通常能提高样本 fidelity，但牺牲 diversity。

问题是，这意味着你要**另外训练一个 classifier**，而且这个 classifier 不能只是普通 clean-image classifier，因为 diffusion sampling 过程中输入的是各种噪声程度的 (x_t)，所以 classifier 也要能处理 noisy images。论文认为这让 pipeline 变得麻烦。

于是作者问：

[  
\boxed{\text{Can we do guidance without a classifier?}}  
]

答案是 yes。

---

# 2. CFG 最关键的思想

CFG 同时需要两个 score：

条件 score

[  
s_c(x_t)=\nabla_{x_t}\log p_t(x_t\mid c),  
]

以及无条件 score

[  
s_u(x_t)=\nabla_{x_t}\log p_t(x_t).  
]

然后把它们做一个线性外插：

# [  
\boxed{  
s_{\mathrm{CFG}}

(1+w)s_c-ws_u  
}  
]

对应论文里的 (\epsilon)-prediction 写法：

# [  
\boxed{  
\tilde\epsilon_\theta(x_t,c)

## (1+w)\epsilon_\theta(x_t,c)

w\epsilon_\theta(x_t)  
}  
]

这就是整篇论文最重要的公式，论文 Eq. (6)。

实际上今天代码里更常看到

# [  
\epsilon_{\rm cfg}

\epsilon_{\rm uncond}  
+s  
(\epsilon_{\rm cond}-\epsilon_{\rm uncond}),  
]

也就是

# [  
\epsilon_{\rm cfg}

s\epsilon_{\rm cond}  
+(1-s)\epsilon_{\rm uncond}.  
]

两种 notation 是一样的，只要令

[  
s=1+w.  
]

所以你在 Stable Diffusion 里看到的 `guidance_scale = 7.5`，通常对应的是后一种 (s) 的定义，而不是论文原始 notation 中的 (w)。

---

# 3. 为什么 conditional − unconditional 可以充当 classifier？

这是这篇 paper 最值得理解的一步。

从 Bayes：

# [  
p(c\mid x_t)

\frac{p(x_t\mid c)p(c)}{p(x_t)}.  
]

取 log：

# [  
\log p(c\mid x_t)

## \log p(x_t\mid c)

\log p(x_t)  
+  
\log p(c).  
]

对 (x_t) 求梯度：

# [  
\nabla_{x_t}\log p(c\mid x_t)

## \nabla_{x_t}\log p(x_t\mid c)

\nabla_{x_t}\log p(x_t).  
]

于是：

# [  
\boxed{  
\nabla\log p(c\mid x_t)

s_c(x_t)-s_u(x_t)  
}  
]

也就是说：

## [  
\boxed{  
\text{conditional score}

\text{unconditional score}  
}  
]

恰好具有一个 **implicit classifier gradient** 的形式。

然后 classifier guidance 原本是

# [  
s_{\rm guided}

s_c+w\nabla\log p(c\mid x_t).  
]

代进去：

# [  
s_{\rm guided}

s_c+w(s_c-s_u),  
]

得到

# [  
\boxed{  
s_{\rm guided}

(1+w)s_c-ws_u.  
}  
]

这就是 CFG。

论文明确把这个解释成通过

[  
p^i(c\mid x_t)  
\propto  
\frac{p(x_t\mid c)}{p(x_t)}  
]

得到的 **implicit classifier**。

---

# 4. 一个非常直观的几何理解

我觉得理解 CFG 最好的方式不是 Bayes，而是看：

# [  
s_{\rm CFG}

s_u+(1+w)(s_c-s_u).  
]

其中

[  
s_c-s_u  
]

可以理解成：

> **“条件 (c) 相对于普通数据分布额外要求你往哪里走？”**

例如 condition 是：

[  
c=\text{“a photo of a golden retriever”}.  
]

那么：

- (s_u)：告诉模型“怎样变得更像一张自然图片”；
    
- (s_c)：告诉模型“怎样变成符合 golden retriever 条件的自然图片”；
    
- (s_c-s_u)：大致就是“golden retriever 这个条件额外提供的方向”。
    

CFG 做的就是把这个方向放大：

[  
\boxed{  
s_u+\lambda(s_c-s_u).  
}  
]

因此它其实是一种 **extrapolation，而不是 interpolation**。

比如：

[  
s_u + 1(s_c-s_u)=s_c  
]

就是普通 conditional model。

但如果：

[  
s_u+7.5(s_c-s_u),  
]

你实际上已经冲过 (s_c) 了，在把 “condition-specific direction” 强行放大。

这就是为什么 guidance scale 太大之后图片往往会出现：

- 颜色过饱和；
    
- 纹理过强；
    
- diversity 降低；
    
- 某些模式变得夸张。
    

论文自己也观察到了强 guidance 会产生更 saturated 的图像。

---

# 5. 那 unconditional model 从哪里来？

这又是 CFG 非常漂亮的地方。

作者**并没有训练两个独立的大模型**。

他们只训练一个：

[  
\epsilon_\theta(x_t,c).  
]

训练的时候，以某个概率 (p_{\rm uncond})，把 condition (c) 丢掉：

[  
c\leftarrow \varnothing.  
]

于是同一个网络有时候学

[  
\epsilon_\theta(x_t,c),  
]

有时候学

[  
\epsilon_\theta(x_t,\varnothing).  
]

后者就是 unconditional prediction：

# [  
\epsilon_{\rm uncond}

\epsilon_\theta(x_t,\varnothing).  
]

论文 Algorithm 1 就只有这个修改：

[  
\boxed{  
c=\varnothing  
\quad\text{with probability }p_{\rm uncond}.  
}  
]

因此训练代码本质上就是：

```python
if random() < p_uncond:
    c = null_condition

eps_pred = model(x_t, t, c)
loss = ||eps - eps_pred||^2
```

极其简单。

在 text diffusion 里，这个 (c=\varnothing) 通常就对应 **empty prompt / null text embedding**。

---

# 6. Sampling 时发生什么？

假设当前状态是 (x_t)。

模型 forward 两次：

# [  
\epsilon_c

\epsilon_\theta(x_t,t,c)  
]

和

# [  
\epsilon_u

\epsilon_\theta(x_t,t,\varnothing).  
]

然后：

# [  
\epsilon_{\rm CFG}

\epsilon_u+s(\epsilon_c-\epsilon_u).  
]

最后把这个新的 (\epsilon_{\rm CFG}) 塑进 DDPM / DDIM / ODE sampler。

所以 CFG 本质上和 sampler 是相对独立的。论文 Algorithm 2 也专门指出 sampling step 可以替换成 DDIM 等其他 sampler。

代价也非常明显：

[  
\boxed{\text{每一个 denoising step 通常要做两次 network evaluation}}  
]

一次 conditional，一次 unconditional。论文也明确指出，因此单纯比较 sampling steps 会有些不公平：256-step CFG 大致需要 (2\times256) 次 denoiser evaluation。

---

# 7. guidance 为什么会“质量↑，diversity↓”？

从 probability distribution 的角度更漂亮。

classifier guidance 对应大致在采：

[  
\tilde p(x\mid c)  
\propto  
p(x\mid c),p(c\mid x)^w.  
]

所以那些 classifier 对 (c) 特别 confident 的区域，会被额外加权。

举个简单的例子。

假设“狗”这个 distribution 很宽：

[  
p(x\mid \text{dog})  
]

包含：

- 各种姿势；
    
- 各种背景；
    
- 模糊的狗；
    
- 奇怪角度的狗；
    
- 非典型的狗。
    

但某些样本 classifier 极其确定：

[  
p(\text{dog}\mid x)\approx1.  
]

增加 (w) 相当于不断偏向这些“最典型、最容易识别”的狗。

所以：

[  
w\uparrow  
]

一般产生：

[  
\text{condition fidelity}\uparrow,  
]

同时

[  
\text{diversity}\downarrow.  
]

这和 GAN 的 truncation / low-temperature sampling 有点类似，也是作者提出 guidance 的出发点之一。

---

# 8. 实验到底证明了什么？

作者主要在 **64×64 和 128×128 ImageNet class-conditional generation** 上验证这个想法。论文的重点不是发明更大的 architecture，而是证明：

[  
\boxed{  
\text{不需要 classifier，也能实现 classifier guidance 类似的 quality–diversity tradeoff}  
}  
]

结果很有代表性：

小量 guidance 时，FID 往往最好；继续加大 guidance，IS 会大幅上升，但 FID 后面会重新变差，同时 diversity 下降。128×128 ImageNet 上，例如 (T=256) 时：

[  
w=0:\quad {\rm FID}=7.27,\quad {\rm IS}=82.45  
]

[  
w=0.3:\quad {\rm FID}=2.43,\quad {\rm IS}=158.47  
]

而

[  
w=4:\quad {\rm FID}=21.53,\quad {\rm IS}=421.03.  
]

这很好地说明：**guidance scale 并不是越大越好，它是在 distribution coverage 和 conditional fidelity 之间移动 operating point。**

另外作者发现 unconditional dropout probability 不需要很大。在他们的 64×64 ImageNet 实验里，

[  
p_{\rm uncond}=0.1,;0.2  
]

明显好于

[  
p_{\rm uncond}=0.5.  
]

也就是说，只需要拿相对少的一部分训练 capacity 去学习 unconditional branch，就足够产生有用的 guidance direction。

---

# 9. 这篇 paper 一个很重要的理论细节

有一个容易被很多教程说得过头的地方。

我们经常说：

# [  
s_c-s_u

\nabla\log p(c\mid x_t).  
]

**如果 (s_c,s_u) 是 exact scores，这是真的。**

但是实际 neural network 学的是

[  
s_\theta(x_t,c),\qquad s_\theta(x_t),  
]

它们只是 approximate score fields，而且 unconstrained neural network 给出的 vector field **未必是 conservative field**。

也就是说，不一定真的存在某个 scalar function (f(x))，满足

# [  
s_\theta(x,c)-s_\theta(x)

\nabla f(x).  
]

所以实际 CFG 并不严格等价于：

> “有一个隐藏 classifier，然后我们正好在走它的 gradient。”

论文特别强调了这一点。implicit classifier 更多是 **motivation / exact-score 情况下的推导**；实际 learned CFG 是直接组合两个 neural score estimates。

这个 distinction 对研究 CFG theory 很重要。

---

# 10. 如果只记住这篇 paper 的三件事

可以压缩成：

# [  
\boxed{  
\epsilon_{\rm cfg}

\epsilon_{\rm uncond}  
+s(\epsilon_{\rm cond}-\epsilon_{\rm uncond})  
}  
]

第一，**training**：

随机把 condition drop 掉：

[  
c\to\varnothing,  
]

让一个网络同时学 conditional / unconditional denoising。

第二，**sampling**：

conditional 和 unconditional 各 forward 一次，然后把

[  
\epsilon_c-\epsilon_u  
]

这个 **condition direction** 放大。

第三，**effect**：

[  
s\uparrow  
\quad\Rightarrow\quad  
\text{condition adherence / fidelity}\uparrow,  
\qquad  
\text{diversity}\downarrow.  
]

所以从今天 diffusion 的视角看，这篇论文真正重要的 insight 可以浓缩成一句话：

> **一个 conditional diffusion model 和它自己的 unconditional prediction 之间的差，就可以充当 guidance signal；没必要再训练外部 classifier。**

而如果从你做 diffusion/flow 研究的角度来看，我觉得这篇最值得继续深挖的其实是

[  
\boxed{s_c-s_u}  
]

这个量：**它究竟在 vector field / probability path / geometry 上代表什么？为什么把它 extrapolate 会提高 perceptual quality？为什么大 CFG 会产生 oversaturation 和 off-manifold behavior？** 这些问题后来催生了大量 CFG rescaling、dynamic CFG、guidance distillation 以及各种 training-free guidance 工作。

如果你愿意，我下一步可以直接从 **score matching / probability flow ODE 的角度**重新推一遍 CFG，并解释为什么

[  
(1+w)s_c-ws_u  
]

可以看成在改变原本的 sampling distribution；这对做 diffusion / flow 的人会比原论文的 Bayes 推导更有意思。
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
> `ingest raw/literature-notes/hoClassifierFreeDiffusionGuidance2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/hoClassifierFreeDiffusionGuidance2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-11T16:51:34.099+08:00 %%
