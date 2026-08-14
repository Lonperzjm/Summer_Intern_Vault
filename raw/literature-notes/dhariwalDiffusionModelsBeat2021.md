---
type: literature-note
citekey: dhariwalDiffusionModelsBeat2021
title: "Diffusion Models Beat GANs on Image Synthesis"
aliases: ["@dhariwalDiffusionModelsBeat2021"]
authors: "Prafulla Dhariwal, Alex Nichol"
firstAuthor: "Dhariwal"
year: 2021
itemType: preprint
doi: "10.48550/arXiv.2105.05233"
url: "http://arxiv.org/abs/2105.05233"
zotero: "zotero://select/library/items/DL5HRFMS"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---artificial-intelligence, computer-science---machine-learning, statistics---machine-learning]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-11
updated: 2026-08-11
ingested_to_wiki: true
wiki_page: "[[wiki/sources/dhariwalDiffusionModelsBeat2021]]"
---

# Diffusion Models Beat GANs on Image Synthesis

> [!info] @dhariwalDiffusionModelsBeat2021 · Dhariwal et al. · 2021
> [Open in Zotero](zotero://select/library/items/DL5HRFMS) · [DOI](https://doi.org/10.48550/arXiv.2105.05233) · [URL](http://arxiv.org/abs/2105.05233) · [PDF](file:///home/lonper/Zotero/storage/NKSLLINE/Dhariwal和Nichol%20-%202021%20-%20Diffusion%20Models%20Beat%20GANs%20on%20Image%20Synthesis.pdf)

## Abstract

> [!abstract]- Click to expand
> We show that diffusion models can achieve image sample quality superior to the current state-of-the-art generative models. We achieve this on unconditional image synthesis by finding a better architecture through a series of ablations. For conditional image synthesis, we further improve sample quality with classifier guidance: a simple, compute-efficient method for trading off diversity for fidelity using gradients from a classifier. We achieve an FID of 2.97 on ImageNet 128$\times$128, 4.59 on ImageNet 256$\times$256, and 7.72 on ImageNet 512$\times$512, and we match BigGAN-deep even with as few as 25 forward passes per sample, all while maintaining better coverage of the distribution. Finally, we find that classifier guidance combines well with upsampling diffusion models, further improving FID to 3.94 on ImageNet 256$\times$256 and 3.85 on ImageNet 512$\times$512. We release our code at https://github.com/openai/guided-diffusion

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 了解一下cfg
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
你给的这篇是 **Dhariwal & Nichol, 2021, _Diffusion Models Beat GANs on Image Synthesis_**。它后来通常也被叫作 **ADM / Guided Diffusion**。这篇论文的历史地位很高：它证明了经过架构改进和 **classifier guidance** 后，diffusion model 在 ImageNet 图像生成上可以在 FID 上超过当时的 BigGAN，同时保持更好的 distribution coverage。([arXiv](https://arxiv.org/abs/2105.05233?utm_source=chatgpt.com "Diffusion Models Beat GANs on Image Synthesis"))

一句话概括：

> **作者一方面把 DDPM 的 U-Net 架构做强，另一方面提出用分类器梯度在采样时“推”diffusion trajectory 朝目标类别走，从而显著提高生成质量。**

---

## 1. 这篇论文想解决什么问题？

在 2021 年之前，DDPM 已经证明 diffusion model 能生成不错的图，但在 ImageNet 这种复杂数据集上，**视觉质量/FID 仍然不如 BigGAN**。

作者认为差距主要来自两个方面：

1. GAN 的网络架构经过多年打磨，而 diffusion 当时的 U-Net 架构还比较朴素；
    
2. GAN 有类似 truncation trick 的东西，可以主动牺牲 diversity 换 fidelity，而 diffusion 缺少这样一个旋钮。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))
    

因此整篇论文其实就是解决这两个问题。

---

# 2. 第一件事：把 diffusion 的 U-Net 做强

基础还是 Ho et al. DDPM：

[  
\epsilon_\theta(x_t,t)  
]

输入 noisy image (x_t) 和 timestep (t)，预测加进去的噪声 (\epsilon)，训练目标还是经典的

# [  
L_{\text{simple}}

\mathbb E\left[  
|\epsilon-\epsilon_\theta(x_t,t)|^2  
\right].  
]

也就是说，**这篇论文并没有重新发明 diffusion 的训练目标**；它首先做的是大量 architecture ablation。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

他们发现有效的改动主要包括：

- 在 (32\times32,16\times16,8\times8) 多个 resolution 加 attention，而不只是 (16\times16)；
    
- multi-head attention，并最终采用约 64 channels/head；
    
- 使用类似 BigGAN 的 residual block 做 up/downsampling；
    
- 使用 **Adaptive Group Normalization (AdaGN)** 注入 timestep 和 class embedding；
    
- 网络适当加宽比单纯加深更加划算。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))
    

AdaGN 写成

# [  
\mathrm{AdaGN}(h,y)

y_s,\mathrm{GroupNorm}(h)+y_b,  
]

其中 (y_s,y_b) 来自 timestep/class embedding 的投影。这个设计后来你在很多 diffusion architecture 里都还能看到类似思想，本质就是一种 FiLM-style conditioning。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

这套改进后的模型，论文称为 **ADM（Ablated Diffusion Model）**。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

---

# 3. 真正最重要的贡献：Classifier Guidance

这才是这篇 paper 最值得认真看的部分。

假设我们已经有一个 diffusion model：

[  
p_\theta(x_t|x_{t+1}),  
]

同时额外训练一个分类器

[  
p_\phi(y|x_t,t).  
]

注意，这个 classifier **不是只在 clean image 上训练**。

它必须能够对每个 noise level 下的 (x_t) 分类，所以 classifier 输入也是 noisy image + timestep。论文明确是在 diffusion 的同一个 noising distribution 上训练 classifier。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

现在我们想从 unconditional distribution

[  
p(x)  
]

变成 class-conditional distribution

[  
p(x|y).  
]

利用 Bayes：

[  
p(x|y)  
\propto  
p(x)p(y|x).  
]

于是取 log gradient：

# [  
\nabla_x\log p(x|y)

\nabla_x\log p(x)  
+  
\nabla_x\log p(y|x).  
]

这就是整个 classifier guidance 的核心。

---

# 4. 从 score 的角度看最清楚

DDPM 的 noise prediction 和 score 有关系：

# [  
\nabla_{x_t}\log p_\theta(x_t)

-\frac{1}{\sqrt{1-\bar\alpha_t}}  
\epsilon_\theta(x_t,t).  
]

加入 classifier 以后：

# [  
\nabla_{x_t}\log p(x_t|y)

\nabla_{x_t}\log p(x_t)  
+  
\nabla_{x_t}\log p_\phi(y|x_t,t).  
]

因此可以把原来的 noise prediction 修改成

# [  
\boxed{  
\hat\epsilon_\theta(x_t,t)

## \epsilon_\theta(x_t,t)

\sqrt{1-\bar\alpha_t}  
\nabla_{x_t}\log p_\phi(y|x_t,t)  
}  
]

然后把这个新的 (\hat\epsilon) 塞回原来的 sampler 里即可。论文对 DDIM 也给出了相应的 guided sampling 形式。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

直觉非常简单：

[  
\underbrace{\text{diffusion score}}_{\text{告诉你哪里像真实图片}}  
+  
\underbrace{\text{classifier gradient}}_{\text{告诉你哪里更像目标类别}}  
]

所以 sampler 每一步不仅问：

> “怎样让这张 noisy image 更像真实图片？”

还额外问：

> “怎样改 (x_t)，才能让 classifier 更确信它是一只狗？”

然后沿这个方向走。

---

# 5. 更精确地说：它修改了 reverse Gaussian 的 mean

论文还有一个很漂亮的推导。

原本 reverse transition 是

# [  
p_\theta(x_t|x_{t+1})

\mathcal N(\mu,\Sigma).  
]

加入条件 (y)：

[  
p(x_t|x_{t+1},y)  
\propto  
p_\theta(x_t|x_{t+1})  
p_\phi(y|x_t).  
]

对 classifier log-probability 在 (\mu) 附近做一阶 Taylor expansion，最后得到

[  
\boxed{  
p(x_t|x_{t+1},y)  
\approx  
\mathcal N(  
\mu+\Sigma g,,  
\Sigma  
)  
}  
]

其中

[  
g=  
\nabla_{x_t}\log p_\phi(y|x_t).  
]

也就是说，**classifier guidance 从概率角度看，就是把 reverse diffusion Gaussian 的 mean 沿 classifier gradient 平移。**([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

这个解释我觉得比“加一个 classifier gradient”更加本质。

---

# 6. Guidance scale (s)：这篇论文最关键的工程发现

作者实际使用的是

# [  
\boxed{  
\mu_{\text{guided}}

\mu  
+  
s\Sigma  
\nabla_{x_t}\log p_\phi(y|x_t)  
}  
]

其中 (s) 就是 **guidance scale**。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

为什么可以乘一个 (s)？

因为

# [  
s\nabla_x\log p(y|x)

\nabla_x\log p(y|x)^s.  
]

所以可以理解成你在 sampling：

[  
p(x)p(y|x)^s.  
]

当

[  
s>1  
]

时，(p(y|x)) 被 sharpen。

结果就是：

[  
\boxed{  
s\uparrow  
\quad\Rightarrow\quad  
\text{fidelity}\uparrow,\qquad  
\text{diversity}\downarrow  
}  
]

这就是 diffusion 版本的 **fidelity–diversity tradeoff knob**。论文实验确实观察到 guidance 增强会提高 precision / class consistency，但过强时 recall 会下降。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

---

# 7. 为什么这件事那么重要？

因为在这之前，你可以把 diffusion 理解成：

[  
x_T\sim\mathcal N(0,I)  
\rightarrow x_{T-1}  
\rightarrow\cdots  
\rightarrow x_0,  
]

网络决定整个 vector field / score field。

Classifier guidance 说明：

[  
\boxed{  
\text{你可以在 sampling 时直接修改这个 score field。}  
}  
]

即

# [  
s_{\rm guided}(x,t)

s_{\rm diffusion}(x,t)  
+  
s,\nabla_x\log p(y|x,t).  
]

这实际上是一个非常通用的思想：

> **pretrained generative prior + external energy / likelihood gradient = conditional generation**

所以从今天的角度看，这篇 paper 的意义远不只是“分类生成”。

你可以把

[  
\log p(y|x)  
]

换成各种 differentiable objective，然后理论上都可以去 modify score。

---

# 8. 它和现在常见的 CFG 有什么关系？

这个非常容易混。

这篇 paper 是：

[  
\boxed{\text{Classifier Guidance}}  
]

需要额外训练 classifier

[  
p_\phi(y|x_t,t).  
]

后来更流行的 **Classifier-Free Guidance（CFG）** 则去掉了外部 classifier，用同一个 diffusion model 同时学 conditional / unconditional prediction。

现代 CFG 常写成

# [  
\epsilon_{\rm CFG}

## \epsilon_{\rm uncond}  
+  
w(  
\epsilon_{\rm cond}

\epsilon_{\rm uncond}  
).  
]

虽然形式不同，但思想是一脉相承的：

[  
\boxed{  
\text{通过放大 conditional direction，牺牲 diversity 换 fidelity。}  
}  
]

因此，理解这篇论文的 classifier guidance，基本就是理解后面 CFG 的一个非常好的前置步骤。

---

# 9. 实验结果有多强？

在 ImageNet class-conditional generation 上，classifier-guided ADM（ADM-G）报告：

[  
\text{FID}_{128}=2.97,  
]

[  
\text{FID}_{256}=4.59,  
]

[  
\text{FID}_{512}=7.72.  
]

当再结合 diffusion upsampler 后，论文报告 (256^2) 的 FID 为 **3.94**，(512^2) 为 **3.85**。论文同时报告 diffusion 相比 BigGAN 有明显更高的 recall，即覆盖更多 data modes。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

例如在 ImageNet (256^2)：

[  
\begin{array}{c|cc}  
&FID&Recall\  
\hline  
\text{BigGAN-deep}&6.95&0.28\  
\text{ADM-G}&4.59&0.52  
\end{array}  
]

所以标题 **“Diffusion Models Beat GANs”** 的依据，不只是生成图看起来不错，而是当时他们在 FID 上超过了 BigGAN，同时 recall 更高。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

---

# 10. 读这篇 paper 时，我建议你抓住这张逻辑图

可以把整篇文章压缩成：

[  
\boxed{\text{DDPM}}  
]

↓

改进 U-Net architecture

[  
\boxed{\text{ADM}}  
]

↓

训练 noisy-image classifier

[  
p_\phi(y|x_t,t)  
]

↓

得到 classifier score

[  
\nabla_{x_t}\log p_\phi(y|x_t,t)  
]

↓

修改 diffusion score

# [  
\boxed{  
\nabla\log p(x_t|y)

\nabla\log p(x_t)  
+  
s\nabla\log p(y|x_t)  
}  
]

↓

[  
\boxed{\text{ADM-G / Guided Diffusion}}  
]

↓

调 (s)

[  
\boxed{\text{diversity}\leftrightarrow\text{fidelity}}  
]

这基本就是这篇 paper **90% 的思想核心**。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

如果你是为了系统学 diffusion，我会特别建议把这篇的 **Section 4 Classifier Guidance** 吃透。它比 Section 3 那些 architecture tricks 更有理论迁移价值：里面的

# [  
\nabla\log p(x|y)

\nabla\log p(x)+\nabla\log p(y|x)  
]

实际上直接把 **score matching、Bayesian conditioning、energy guidance 和后来的 CFG** 串起来了。论文自己也强调 classifier gradient 可以作为一种通用方式来改变预训练 diffusion model 的条件生成行为。([ar5iv](https://ar5iv.labs.arxiv.org/html/2105.05233 "[2105.05233] Diffusion Models Beat GANs on Image Synthesis"))

如果你愿意，我下一步可以直接给你**从 DDPM 的 (q(x_t|x_0)) 开始，把论文 Section 4 的 classifier guidance 公式一步一步完整推出来**，包括为什么最后 mean 会变成  
[  
\mu+\Sigma\nabla_x\log p(y|x),  
]  
这个推导很值得掌握。
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
> `ingest raw/literature-notes/dhariwalDiffusionModelsBeat2021.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/dhariwalDiffusionModelsBeat2021.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-11T16:22:52.557+08:00 %%
