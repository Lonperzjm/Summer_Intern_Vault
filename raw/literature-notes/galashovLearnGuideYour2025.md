---
type: literature-note
citekey: galashovLearnGuideYour2025
title: "Learn to Guide Your Diffusion Model"
aliases: ["@galashovLearnGuideYour2025"]
authors: "Alexandre Galashov, Ashwini Pokle, Arnaud Doucet, Arthur Gretton, Mauricio Delbracio, Valentin De Bortoli"
firstAuthor: "Galashov"
year: 2025
itemType: preprint
doi: "10.48550/ARXIV.2510.00815"
url: "https://arxiv.org/abs/2510.00815"
zotero: "zotero://select/library/items/A68P56H4"
tags: [literature, fos:-computer-and-information-sciences, machine-learning-(cs.lg), machine-learning-(stat.ml)]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-14
updated: 2026-08-14
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/galashovLearnGuideYour2025]]" # e.g. "[[wiki/sources/galashovLearnGuideYour2025]]"
---

# Learn to Guide Your Diffusion Model

> [!info] @galashovLearnGuideYour2025 · Galashov et al. · 2025
> [Open in Zotero](zotero://select/library/items/A68P56H4) · [DOI](https://doi.org/10.48550/ARXIV.2510.00815) · [URL](https://arxiv.org/abs/2510.00815) · [PDF](file:///home/lonper/Zotero/storage/3U7CF2QR/Galashov%20等%20-%202025%20-%20Learn%20to%20Guide%20Your%20Diffusion%20Model.pdf)

## Abstract

> [!abstract]- Click to expand
> Classifier-free guidance (CFG) is a widely used technique for improving the perceptual quality of samples from conditional diffusion models. It operates by linearly combining conditional and unconditional score estimates using a guidance weight ω. While a large, static weight can markedly improve visual results, this often comes at the cost of poorer distributional alignment. In order to better approximate the target conditional distribution, we instead learn guidance weights ωc,(s,t), which are continuous functions of the conditioning c, the time t from which we denoise, and the time s towards which we denoise. We achieve this by minimizing the distributional mismatch between noised samples from the true conditional distribution and samples from the guided diffusion process. We extend our framework to reward guided sampling, enabling the model to target distributions tilted by a reward function R(x0, c), defined on clean data and a conditioning c. We demonstrate the effectiveness of our methodology on low-dimensional toy examples and highdimensional image settings, where we observe improvements in Fréchet inception distance (FID) for image generation. In text-to-image applications, we observe that employing a reward function given by the CLIP score leads to guidance weights that improve image-prompt alignment.

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
这篇是 **《Learn to Guide Your Diffusion Model》**，核心想法可以压成一句话：

> **不要再手调 CFG guidance scale，而是把 guidance weight 本身学出来，让它随 diffusion time 和 condition/prompt 自适应变化。** ([arXiv](https://arxiv.org/abs/2510.00815 "[2510.00815] Learn to Guide Your Diffusion Model"))

而且它的切入点挺有意思：作者不是把 CFG 理解成“我要故意改变目标分布，让 condition 更强”，而更倾向于把 CFG 看成 **对 imperfect denoiser 的 correction**。因此，如果 denoiser 的误差随时间、类别、prompt 都不同，那么固定的 CFG scale 本来就没有理由是最优的。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

## 1. 它想解决什么问题？

标准 CFG 写成论文里的形式大概是

# [  
\hat x_\theta(x_t,c;\omega)

\hat x_\theta(x_t,c)  
+  
\omega\left[  
\hat x_\theta(x_t,c)-\hat x_\theta(x_t,\varnothing)  
\right].  
]

通常我们会人为选一个固定的 (\omega)，或者设计一个随 (t) 变化的 schedule。

问题是：

- 不同 diffusion timestep 需要的 guidance 强度可能不同；
    
- 不同 class / prompt 需要的 guidance 也可能不同；
    
- 大 CFG 往往提高 perceptual quality / prompt alignment，但会损害 distributional fidelity；
    
- 最关键的是，真实网络 (\hat x_\theta) 只是  
    [  
    \mathbb E[x_0|x_t,c]  
    ]  
    的近似，因此 CFG 很可能实际上是在**纠正 denoiser approximation error**。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))
    

所以作者直接定义

[  
\boxed{  
\omega=\omega_\phi(c,s,t)  
}  
]

其中：

- (c)：condition/class/prompt；
    
- (t)：当前 noise level；
    
- (s<t)：要从 (t) denoise 到的目标 noise level。
    

也就是说，**每个 condition、每一步 denoising 都允许拥有不同 guidance strength。** ([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

# 2. 最核心的 trick：self-consistency

这是整篇论文最值得看的东西。

给一个真实训练样本 (x_0)，考虑两条到达 noise level (s) 的路径。

### 路径 A：直接 forward noise

直接把 (x_0) 加噪到 (s)：

[  
x_s\sim p_{s|0}(x_s|x_0).  
]

这是我们知道的 ground-truth forward process。

### 路径 B：先加更多噪声，再用模型 denoise 回来

先

[  
x_0\rightarrow x_t,\qquad t>s,  
]

然后用带 learnable CFG 的 frozen diffusion model

[  
x_t  
\overset{\omega_\phi(c,s,t)}{\longrightarrow}  
\tilde x_s.  
]

如果 reverse model 是完美的，那么理论上这两条路径得到的 (x_s) 应当服从同一个 distribution：

[  
\boxed{  
p(\tilde x_s|x_0,c)  
\approx  
p(x_s|x_0)  
}  
]

作者把这个叫 **self-consistency condition**。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

直观上就是：

[  
\boxed{  
x_0  
\xrightarrow{\text{noise to }t}  
x_t  
\xrightarrow{\text{learned guided denoise}}  
x_s  
}  
]

应该和

[  
\boxed{  
x_0  
\xrightarrow{\text{noise directly to }s}  
x_s  
}  
]

统计上对得上。

这个视角其实非常漂亮，因为你**不需要知道 optimal guidance scale 的标签**。

训练数据本身就给出了 supervision。

---

# 3. 怎么衡量两个 distribution 对没对上？

作者用的是 **MMD / energy distance 类目标**。

定义 guided sample (\tilde x_s(\omega))，target sample 为 (x_s)，他们优化类似

# [  
\mathcal L_{\beta,\lambda}

## \mathbb E  
\left[  
|\tilde x_s-x_s|^\beta

\frac{\lambda}{2}  
|\tilde x_s-\tilde x'_s|^\beta  
\right].  
]

第一项：

[  
|\tilde x_s-x_s|^\beta  
]

把 generated distribution 往 target 拉。

第二项：

[  
-|\tilde x_s-\tilde x_s'|^\beta  
]

避免所有 proposal collapse 到同一点，相当于保留 distributional spread。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

所以它不是简单要求

[  
\tilde x_s=x_s,  
]

而是要求

[  
\boxed{  
p_\phi(\tilde x_s|x_0,c)  
\simeq  
p(x_s|x_0).  
}  
]

这是它和普通 regression-style guidance learning 很重要的区别。

---

## 4. 它甚至有一个特别简单的 (L_2) 版本

令

[  
\beta=2,\qquad\lambda=0,  
]

就得到

# [  
\boxed{  
\mathcal L_{L_2}

\mathbb E|\tilde x_s-x_s|^2.  
}  
]

作者发现这个版本也相当有效，而且比完整 MMD 便宜，因为完整版本需要 particle-particle interaction，复杂度有 (O(m^2))；不过 (L_2) 对 hyperparameter 更敏感。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

# 5. 实际训练时什么被更新？

这里很重要：

**原来的 diffusion / flow model 完全 freeze。**

只训练一个小 guidance network：

[  
\boxed{  
\omega_\phi(s,t,c).  
}  
]

过程大致是：

[  
(x_0,c)  
]

采样两个时间

[  
s<t,  
]

得到

[  
x_s\sim q_s(x_s|x_0),  
\qquad  
x_t\sim q_t(x_t|x_0),  
]

然后

[  
x_t  
\xrightarrow{  
\text{frozen denoiser + }\omega_\phi  
}  
\tilde x_s  
]

最后更新

[  
\phi  
]

让

[  
p(\tilde x_s|x_0,c)  
\approx  
p(x_s|x_0).  
]

算法本身基本就是这么简单。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

一个挺有意思的 empirical observation 是：虽然 inference 时相邻 sampling steps 的

[  
|t-s|  
]

往往很小，作者反而发现**训练 guidance network 时使用较大的 time gap（约 (\delta\sim0.1)）效果更好**。他们猜测更大的 gap 能提供更稳定、更 informative 的 gradient signal。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

# 6. 它学出来的 CFG 到底长什么样？

一个很重要的发现是：

[  
\boxed{  
\text{optimal guidance 不仅随 }t\text{ 变化，还强烈随 condition 变化。}  
}  
]

例如 ImageNet 中，不同类别学出来的 guidance curve 差别很大：

- 有些 class 基本不需要 guidance；
    
- 有些 class 在很宽的 timestep 区间需要比较强的 guidance。
    

作者甚至举例，ImageNet 的 prairie chicken 类学出的 guidance 几乎是 (0)，而 paintbrush 类明显更 aggressive。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

这实际上支持了论文最初的 hypothesis：

> CFG 不应该被看成一个 global hyperparameter，而更像是一个 **conditional correction field**。

我觉得这是论文最有启发性的结果之一。

---

# 7. 实验效果如何？

### ImageNet (64\times64)

FID：

|方法|FID ↓|
|---|--:|
|无 CFG|4.46|
|constant CFG|2.40|
|Limited Interval Guidance|2.11|
|**Learned self-consistency CFG**|**1.99**|

也就是说，它确实比精心 grid-search 的 guidance schedule 还进一步降了一些 FID。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

CelebA 上也是类似：

[  
2.44;(\text{unguided})  
\rightarrow  
2.37;(\text{LIG})  
\rightarrow  
\boxed{2.10};(\text{learned}).  
]

([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

# 8. 对 text-to-image 也做了

作者还在 MS-COCO (512^2) 上训练了一个 **1.05B flow-matching model**，然后 freeze generator，只学习 prompt-dependent guidance：

[  
\omega_\phi(c_{\rm CLIP},c_{\rm T5},s,t).  
]

([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

结果挺有意思：

|方法|FID ↓|CLIP ↑|
|---|--:|--:|
|Unguided|24.74|0.278|
|Fixed CFG (7.5)|31.20|**0.306**|
|Learned guidance|**18.01**|0.295|
|Learned + CLIP reward|28.37|**0.306**|

([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

这个表非常说明问题：

传统大 CFG：

[  
\text{alignment}\uparrow,\qquad  
\text{distribution quality}\downarrow.  
]

而 self-consistency learned guidance 找到了一个更好的 fidelity/alignment tradeoff。

如果再显式加入 CLIP reward：

# [  
\mathcal L

\mathcal L_{\rm consistency}  
+  
\gamma_R\mathcal L_{\rm reward},  
]

就可以把 CLIP alignment 再推上去。作者特意用 consistency loss regularize reward optimization，以减轻纯 reward optimization 导致 reward hacking 的问题。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

---

# 9. 从 diffusion / flow 的角度，我会怎么理解它

我认为最有价值的不是“又一种 CFG scheduler”，而是下面这个 interpretation：

理想 reverse dynamics 应该满足一种 Chapman–Kolmogorov / marginal consistency 式结构：

# [  
q_s

q_tP_{t\rightarrow s}.  
]

但 learned reverse kernel

[  
P^\theta_{t\rightarrow s}  
]

因为 score/denoiser approximation error 不满足它。

CFG 相当于引入一个很低维的 correction：

[  
P^\theta_{t\rightarrow s}  
\quad\longrightarrow\quad  
P^{\theta,\omega}_{t\rightarrow s},  
]

然后通过 distribution matching 求

# [  
\boxed{  
\omega^*

\arg\min_\omega  
D\left(  
q_tP^{\theta,\omega}_{t\rightarrow s},  
q_s  
\right).  
}  
]

这基本就是整篇 paper 的数学本质。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

所以你也可以把它理解为：

[  
\boxed{  
\text{learning a low-dimensional correction to an imperfect reverse transport}  
}  
]

而不是单纯“学习 CFG scale”。

---

# 10. 我觉得这篇的优点和局限

**优点**是 idea 很干净：base model 不用 retrain，有真实数据就能 self-supervise guidance，而且 learned (\omega(c,s,t)) 的确发现了明显的 condition dependence。它还能自然推广到 reward-guided generation。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

但理论上还没有完全闭环。作者自己也明确说，self-consistency condition **比真正需要的 marginal consistency 强得多**，而且由于 (\omega) 只依赖 (c,s,t)，不依赖当前 state (x_t)，一般不可能严格满足这个条件；它本质上是一个实用的 surrogate learning signal。作者也把“理论理解 objective 以及 resulting guidance solution”列为 future work。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))

所以我会把这篇文章概括成：

[  
\boxed{  
\text{把 CFG 从一个手调超参数，提升成一个通过 distributional consistency 学习的 adaptive control variable。}  
}  
]

如果只读 **3 个地方**，我建议重点看 **Eq. (19) self-consistency → Eq. (22) MMD objective → Table 1/3**。这三处基本就把论文的思想、方法和效果串起来了。([arXiv](https://arxiv.org/html/2510.00815v1 "Learn to Guide Your Diffusion Model"))
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
> `ingest raw/literature-notes/galashovLearnGuideYour2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/galashovLearnGuideYour2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-14T14:38:49.928+08:00 %%
