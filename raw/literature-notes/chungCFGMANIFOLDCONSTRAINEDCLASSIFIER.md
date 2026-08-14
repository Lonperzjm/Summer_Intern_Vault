---
type: literature-note
citekey: chungCFGMANIFOLDCONSTRAINEDCLASSIFIER
title: "CFG++: MANIFOLD-CONSTRAINED CLASSIFIER FREE GUIDANCE FOR DIFFUSION MODELS"
aliases:
  - "@chungCFGMANIFOLDCONSTRAINEDCLASSIFIER"
authors: Hyungjin Chung, Jeongsol Kim, Geon Yeong Park, Hyelin Nam, Jong Chul Ye
firstAuthor: Chung
year: 2024
itemType: journalArticle
zotero: zotero://select/library/items/L8XVTGGM
tags:
  - literature
status: unread
priority: P2
my-rating:
created: 2026-08-14
updated: 2026-08-14
ingested_to_wiki: true
wiki_page: "[[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]"
---

# CFG++: MANIFOLD-CONSTRAINED CLASSIFIER FREE GUIDANCE FOR DIFFUSION MODELS

> [!info] @chungCFGMANIFOLDCONSTRAINEDCLASSIFIER · Chung et al. · Error: `format` can only be applied to dates. Tried for format object
> [Open in Zotero](zotero://select/library/items/L8XVTGGM) · [PDF](file:///home/lonper/Zotero/storage/IWNECBYL/Chung%20等%20-%20CFG++%20MANIFOLD-CONSTRAINED%20CLASSIFIER%20FREE%20GUIDANCE%20FOR%20DIFFUSION%20MODELS.pdf)

## Abstract

> [!abstract]- Click to expand
> 

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
-
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

### Imported 2026-08-14 13:55

- 🟡 **p.2** — 1 INTRODUCTION [⤴](zotero://open-pdf/library/items/IWNECBYL?page=2&annotation=YA9ZELXA)
  - 💬 *我的批注*：本质让w随时间变

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
这篇 **CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models** 的核心其实很简单：

> **传统 CFG 的很多问题，不一定是 diffusion 本身的问题，而是 CFG 把采样轨迹推到了 data manifold 外面。CFG++ 通过一个很小的 sampling update 改动，让 text guidance 只作用在 denoising 部分，而 renoising/transport 部分仍然使用 unconditional diffusion 的方向。** ([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

这个改动几乎**没有额外计算量**，但作者发现它能同时改善生成质量、DDIM inversion、image editing 和 inverse problem。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

---

## 1. 先说传统 CFG 在干什么

设

[  
\epsilon_u(x_t)=\epsilon_\theta(x_t,\varnothing),  
\qquad  
\epsilon_c(x_t)=\epsilon_\theta(x_t,c).  
]

普通 classifier-free guidance 用

# [  
\epsilon_{\rm CFG}

\epsilon_u  
+\omega(\epsilon_c-\epsilon_u).  
]

如果 (\omega=1)，就是普通 conditional model；实际 Stable Diffusion 里往往要用

[  
\omega\gg 1,  
]

比如 5、7.5、10 甚至更大，才能有比较强的 prompt adherence。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

问题在于，这从几何上实际上是一个 **extrapolation**：

# [  
\hat x_{0,\rm CFG}

(1-\omega)\hat x_{0,u}  
+\omega \hat x_{0,c}.  
]

当

[  
0\leq \omega\leq 1  
]

时，你是在 unconditional prediction 和 conditional prediction 之间**插值**。

但是当

[  
\omega>1  
]

时，你已经越过 conditional prediction，沿着

[  
\hat x_{0,c}-\hat x_{0,u}  
]

继续往外走。

作者的观点是：

> **这个 extrapolation 很容易把 (\hat x_0) 推离 learned data manifold。**

于是产生 saturation、artifact、mode collapse、diversity 下降等现象。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

---

# 2. 这篇文章最关键的 insight：CFG 的问题有两个

作者认为传统 CFG 会以 **两种方式 off-manifold**。

第一种刚才讲了：

[  
\omega>1  
]

导致 denoised estimate extrapolation。

但第二种其实更关键，也正是 CFG++ 真正修改的地方：

**CFG 不仅用 guided score 做 denoising，还用 guided score 做下一步的 renoising。** ([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

以 DDIM 为例。

普通 unconditional DDIM 大概是

# [  
\hat x_0

\frac{  
x_t-\sqrt{1-\bar\alpha_t}\epsilon_u  
}{  
\sqrt{\bar\alpha_t}  
},  
]

然后

# [  
x_{t-1}

\sqrt{\bar\alpha_{t-1}}\hat x_0  
+  
\sqrt{1-\bar\alpha_{t-1}}\epsilon_u.  
]

可以把它理解成：

[  
x_t  
\rightarrow  
\underbrace{\hat x_0}_{\text{denoise}}  
\rightarrow  
\underbrace{x_{t-1}}_{\text{renoise}}.  
]

而传统 CFG 把这两个地方都换成

[  
\epsilon_{\rm CFG}.  
]

也就是

[  
x_t  
\stackrel{\epsilon_{\rm CFG}}{\longrightarrow}  
\hat x_{0,\rm CFG}  
\stackrel{\epsilon_{\rm CFG}}{\longrightarrow}  
x_{t-1}.  
]

作者认为第二步也用 heavily guided 的方向，会进一步把 (x_{t-1}) 推离正确的 noisy manifold (\mathcal M_{t-1})。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

---

# 3. CFG++ 的改动简单到有点离谱

他们仍然计算一个 guidance：

# [  
\epsilon_\lambda

\epsilon_u  
+  
\lambda(\epsilon_c-\epsilon_u),  
]

但是只让

[  
\lambda\in[0,1].  
]

然后用它得到 conditional denoised estimate：

# [  
\hat x_{0,c}^{\lambda}

\frac{  
x_t-\sqrt{1-\bar\alpha_t}\epsilon_\lambda  
}{  
\sqrt{\bar\alpha_t}  
}.  
]

**关键来了。**

传统 CFG：

# [  
\boxed{  
x_{t-1}

\sqrt{\bar\alpha_{t-1}}\hat x_{0,c}^{\omega}  
+  
\sqrt{1-\bar\alpha_{t-1}}  
\epsilon_{\rm CFG}  
}  
]

CFG++：

# [  
\boxed{  
x_{t-1}

\sqrt{\bar\alpha_{t-1}}\hat x_{0,c}^{\lambda}  
+  
\sqrt{1-\bar\alpha_{t-1}}  
\epsilon_u  
}  
]

看到区别了吗？

只有最后一项不同：

[  
\color{red}{\epsilon_{\rm CFG}}  
\quad\longrightarrow\quad  
\color{blue}{\epsilon_u}.  
]

也就是说：

> **conditional guidance 负责决定“我要往哪个 clean image 走”；而从 clean estimate 返回 noisy manifold 的 transport/renoising，则交还给原始 unconditional diffusion model。** ([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

这基本就是整篇 paper 最重要的公式。

---

# 4. 为什么这个东西叫 “manifold-constrained”？

作者是从 diffusion inverse problem 的角度推出来的，而不是拍脑袋改 sampler。

他们把 text guidance 重新理解成：

[  
\min_{x\in\mathcal M} \ell_{\rm SDS}(x;c),  
]

其中

# [  
\ell_{\rm SDS}(x)

|  
\epsilon_\theta(  
\sqrt{\bar\alpha_t}x  
+  
\sqrt{1-\bar\alpha_t}\epsilon,  
c  
)  
-\epsilon  
|^2.  
]

也就是说：

> 我们不是想直接构造一个奇怪的 sharpened distribution；我们是在 **data manifold (\mathcal M) 上找一个最符合 text condition 的样本**。

然后作者借用了 diffusion inverse solver 里的 **DDS / decomposed diffusion sampling** 思想：先让 diffusion model 把你保持在 manifold 附近，再沿 condition loss 做 constrained correction。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

推导以后居然正好得到：

[  
\hat x_{0,u}  
+  
\lambda(  
\hat x_{0,c}-\hat x_{0,u}  
),  
]

然后 renoise 用

[  
\epsilon_u.  
]

所以 CFG++ 可以理解为：

[  
\boxed{  
\text{unconditional manifold transport}  
+  
\text{conditional optimization}  
}  
]

而不是普通 CFG 那种：

[  
\boxed{  
\text{直接把整个 diffusion vector field 都强行 extrapolate}  
}  
]

这是这篇论文比较漂亮的地方。

---

# 5. 为什么 (\lambda\leq1) 反而可以达到 CFG (\omega=10+) 的效果？

这个一开始非常反直觉。

CFG++ 里

[  
\lambda\in[0,1]  
]

只是 interpolation：

# [  
\hat x_{0,\lambda}

(1-\lambda)\hat x_{0,u}  
+  
\lambda\hat x_{0,c}.  
]

你可能会想：

> 那 guidance 岂不是特别弱？

但作者的解释是，CFG 之所以以前需要 (\omega\gg1)，部分原因恰恰是它的 sampling dynamics 本身有问题。

CFG++ 每一步都在显式降低 text-conditioned score-matching/SDS loss：

[  
|x-\hat x_{0,c}|^2,  
]

所以连续很多步的小 correction 会逐渐把 trajectory 带向 text-conditioned region，而不需要每一步暴力 extrapolate。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

他们实验上找出的近似 correspondence 很有意思：

[  
\begin{array}{c|ccccc}  
\text{CFG }\omega  
&2&5&7.5&9&12.5\  
\hline  
\text{CFG++ }\lambda  
&0.2&0.4&0.6&0.8&1.0  
\end{array}  
]

这是在 SD v1.5、50-step DDIM 下按照生成结果的 LPIPS 匹配出来的，不是一个普适解析映射。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

所以

[  
\boxed{\lambda=1}  
]

在他们那个 setting 里视觉效果大致已经对应普通

[  
\boxed{\omega\simeq12.5}.  
]

---

# 6. 它为什么能改善 DDIM inversion？

这一点其实对理解论文特别重要。

DDIM inversion 依赖一个近似：

[  
\epsilon(x_t)\approx\epsilon(x_{t-1}).  
]

普通 unconditional diffusion 里 timestep 很小时，这个假设通常还可以。

但是 CFG 是

# [  
\epsilon_{\rm CFG}

\epsilon_u+  
\omega(\epsilon_c-\epsilon_u).  
]

于是它的 timestep-to-timestep error 里会出现

## [  
\omega  
\left[  
\delta\epsilon_c(x_t)

\delta\epsilon_c(x_{t-1})  
\right].  
]

也就是说 guidance scale (\omega) **直接把 inversion error 放大**。

而 CFG++ 对应误差只有

## [  
\lambda  
\left[  
\delta\epsilon_c(x_t)

\delta\epsilon_c(x_{t-1})  
\right],  
]

又因为通常

[  
\lambda\ll \omega,  
]

所以 inversion error 小很多。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

这解释了为什么传统 CFG 做 real-image inversion/editing 很麻烦，而 CFG++ 可以接近 DDIM 本来的 invertibility。

---

# 7. 实验上怎么样？

作者主要展示了三类结果。

**Text-to-image。** 在 matched guidance strength 下，CFG++ 基本一直获得更低 FID，同时 CLIP alignment 持平或略好。例如 SD v1.5、50 NFE DDIM、COCO 10k 上，在对应 ((\omega,\lambda)=(2,0.2)) 时，FID 从 13.84 降到 12.75；其他 guidance scale 上也有类似趋势。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

**DDIM inversion / editing。** 普通 CFG guidance 增大后 reconstruction 很快恶化，而 CFG++ across guidance scales 都明显更稳定；做 text editing 时，也更容易保留原图里的 background / scene structure。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

**Inverse problems。** 作者还把它接到 latent diffusion inverse solver PSLD 上做 super-resolution、deblurring、inpainting，证明 text guidance 不再像 vanilla CFG 那样容易导致 solver divergence。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

另外它不是 DDIM-only trick。作者也说明了怎么推广到 Euler、Euler ancestral、DPM-Solver++ 等高阶 solver，以及 SDXL-Lightning / Turbo 一类 distilled model。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

---

# 8. 从 diffusion / flow 的角度，我觉得最值得注意的是这一点

如果把 diffusion sampler 理解成一个时间依赖 vector field

[  
\frac{dx_t}{dt}=v_t(x_t),  
]

传统 CFG 基本是在干

# [  
v_t^{\rm CFG}

v_t^u  
+  
\omega(v_t^c-v_t^u).  
]

问题是当

[  
\omega\gg1  
]

时，这个 vector field 未必还对应一个“合理的 diffusion transport direction”。事实上这篇文章引用的相关工作也指出，CFG score 本身可能不是一个 valid denoising direction。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

CFG++ 的视角更像是在把一步 dynamics 拆成：

[  
\boxed{  
\text{transport along learned generative dynamics}  
}  
]

和

[  
\boxed{  
\text{condition-induced correction}  
}  
]

然后**只对后者做 guidance**。

我觉得这是比“CFG++ 多了一个技巧”更值得记住的 conceptual takeaway：

[  
\boxed{  
\text{guidance does not necessarily need to modify the whole generative vector field.}  
}  
]

它可以被理解成一种 **operator splitting / projected dynamics / manifold-constrained correction**。

这个视角其实非常适合拿来思考 flow matching、rectified flow 以及一般 conditional generative ODE 的 guidance：与其直接做

[  
v_{\rm guided}=v_u+\omega(v_c-v_u),  
]

是否可以把 **base transport dynamics** 和 **conditional steering dynamics** 分开处理？

---

## 9. 这篇论文我认为最应该记住的 3 个式子

普通 CFG：

# [  
\epsilon_{\rm CFG}

\epsilon_u+\omega(\epsilon_c-\epsilon_u),  
\qquad \omega>1.  
]

CFG++ 的 guided clean prediction：

# [  
\hat x_{0,\lambda}

(1-\lambda)\hat x_{0,u}  
+  
\lambda \hat x_{0,c},  
\qquad 0\leq\lambda\leq1.  
]

最关键的 sampling：

# [  
\boxed{  
x_{t-1}

\sqrt{\bar\alpha_{t-1}}\hat x_{0,\lambda}  
+  
\sqrt{1-\bar\alpha_{t-1}}\epsilon_u  
}  
]

**一句话理解：**

[  
\boxed{\text{Guide the denoising, not the renoising.}}  
]

不过有一点要区分：论文里的“manifold”解释依赖 piecewise-linear/local manifold 等假设，所以它更像一个**有理论动机、且实验支持很强的几何解释**，而不是对任意真实数据 manifold 都严格成立的定理。([arXiv](https://arxiv.org/html/2406.08070v2 "CFG++: Manifold-constrained Classifier Free Guidance for Diffusion Models"))

如果你想继续深入，我建议下一步直接看它 **Eq. (9) (\rightarrow) Eq. (13)** 的推导。这几行基本就是全篇的理论核心，而且里面 **SDS / DDS → CFG++** 的联系很有意思。
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：
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
> `ingest raw/literature-notes/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-14T13:56:34.921+08:00 %%
