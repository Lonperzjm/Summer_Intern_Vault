---
type: literature-note
citekey: kynkaanniemiApplyingGuidanceLimited2024
title: "Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"
aliases: ["@kynkaanniemiApplyingGuidanceLimited2024"]
authors: "Tuomas Kynkäänniemi, Miika Aittala, Tero Karras, Samuli Laine, Timo Aila, Jaakko Lehtinen"
firstAuthor: "Kynkäänniemi"
year: 2024
itemType: preprint
doi: "10.48550/arXiv.2404.07724"
url: "http://arxiv.org/abs/2404.07724"
zotero: "zotero://select/library/items/LYRB5AG5"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---artificial-intelligence, computer-science---machine-learning, statistics---machine-learning, computer-science---neural-and-evolutionary-computing]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-11
updated: 2026-08-11
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024]]" # e.g. "[[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024]]"
---

# Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models

> [!info] @kynkaanniemiApplyingGuidanceLimited2024 · Kynkäänniemi et al. · 2024
> [Open in Zotero](zotero://select/library/items/LYRB5AG5) · [DOI](https://doi.org/10.48550/arXiv.2404.07724) · [URL](http://arxiv.org/abs/2404.07724) · [PDF](file:///home/lonper/Zotero/storage/9E9WHNBZ/Kynkäänniemi%20等%20-%202024%20-%20Applying%20Guidance%20in%20a%20Limited%20Interval%20Improves%20Sample%20and%20Distribution%20Quality%20in%20Diffusion%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Guidance is a crucial technique for extracting the best performance out of image-generating diffusion models. Traditionally, a constant guidance weight has been applied throughout the sampling chain of an image. We show that guidance is clearly harmful toward the beginning of the chain (high noise levels), largely unnecessary toward the end (low noise levels), and only beneficial in the middle. We thus restrict it to a specific range of noise levels, improving both the inference speed and result quality. This limited guidance interval improves the record FID in ImageNet-512 significantly, from 1.81 to 1.40. We show that it is quantitatively and qualitatively beneficial across different sampler parameters, network architectures, and datasets, including the large-scale setting of Stable Diffusion XL. We thus suggest exposing the guidance interval as a hyperparameter in all diffusion models that use guidance.

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
这篇论文的核心非常简单，但很有启发性：

> **CFG / guidance 不应该从采样开始一直开到结束。它真正有用的只是中间一段 noise level。**

论文题目是 **“Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models”**，作者包括 Tuomas Kynkäänniemi、Tero Karras、Timo Aila、Jaakko Lehtinen 等，发表于 NeurIPS 2024。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

### 1. 它发现了什么？

传统 classifier-free guidance（CFG）基本可以抽象成

# [  
s_{\text{guided}}(x_t,t)

s_{\text{cond}}(x_t,t)  
+  
w\big(s_{\text{cond}}(x_t,t)-s_{\text{uncond}}(x_t,t)\big),  
]

或者在 (\epsilon)-prediction / denoiser parameterization 下写成对应形式。

通常做 sampling 时，会选一个固定 (w)，然后从

[  
t=T\quad\longrightarrow\quad t=0  
]

**每一步都做 guidance**。

这篇文章问了一个很直接的问题：

**CFG 真的每个 timestep 都有用吗？**

他们实验发现不是，而是呈现出一个非常明显的“三阶段”结构：([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

[  
\boxed{  
\text{high noise: harmful}  
\quad\to\quad  
\text{middle noise: useful}  
\quad\to\quad  
\text{low noise: unnecessary}  
}  
]

也就是：

|Sampling 阶段|noise (\sigma)|Guidance 效果|
|---|--:|---|
|刚开始|很高|**有害**|
|中间|中等|**最有用**|
|快结束|很低|**基本没用**|

所以他们提出：

[  
w(\sigma)=  
\begin{cases}  
0, & \sigma>\sigma_{\max}\  
w, & \sigma_{\min}\leq\sigma\leq\sigma_{\max}\  
0, & \sigma<\sigma_{\min}  
\end{cases}  
]

概念上就是：

```text
noise
high -------------------------------------- low

       no CFG       CFG ON        no CFG
     |----------|=============|----------|
         early       middle        late
```

这就是论文所谓的 **Guidance Interval / Limited Interval Guidance**。

---

## 2. 为什么 high-noise 阶段做 CFG 反而有害？

这是这篇论文最值得关注的观察。

在 sampling 初期，

[  
x_t \approx \text{noise},  
]

图像的 global structure / semantic choice 还没有真正确定。

此时 conditional 和 unconditional prediction 的差异，被 CFG 用较大的 guidance scale 放大。

结果就是：模型在**过早阶段就被强迫做 semantic decision**。

直觉上，比如 ImageNet 类别是：

> “golden retriever”

原本无 guidance 时，生成 distribution 可能覆盖很多姿势、背景、构图、外观。

但如果 high-noise 时就强 guidance，相当于在图像还只有很粗的 low-frequency structure 时就把 trajectory 推向少数几个特别符合 class condition 的区域。

因此会造成：

[  
\text{distribution truncation}  
]

即 **mode coverage / diversity 下降**。

论文明确观察到，把过高的 noise level 纳入 guidance interval 会导致 image distribution 被截断，并恶化 FID。([NeurIPS Proceedings](https://proceedings.neurips.cc/paper_files/paper/2024/file/dd540e1c8d26687d56d296e64d35949f-Paper-Conference.pdf?utm_source=chatgpt.com "Applying Guidance in a Limited Interval Improves Sample ..."))

这实际上解释了 CFG 长期以来一个熟悉的 trade-off：

[  
\text{higher CFG}  
\Rightarrow  
\begin{cases}  
\text{better condition fidelity}\  
\text{worse diversity}  
\end{cases}  
]

作者的观点可以理解成：

> 问题不一定只是 guidance scale 太大，而是 **你在错误的时间用了 guidance**。

这是这篇工作的核心 insight。

---

## 3. 为什么最后阶段也不需要 guidance？

sampling 后期：

[  
\sigma\rightarrow 0.  
]

这时图像的

- class identity
    
- pose
    
- global layout
    
- composition
    

基本已经确定了。

剩下更多是在恢复局部细节、高频纹理。

而此时 conditional model 和 unconditional model 给出的 denoising direction 会越来越相似，因此

[  
s_{\mathrm{cond}}-s_{\mathrm{uncond}}  
]

贡献已经很小。

换句话说：

[  
\underbrace{\text{guidance}}_{\text{semantic steering}}  
]

已经完成任务了。

后面的 denoising 再算 unconditional branch，收益非常有限。论文因此认为 low-noise 区域的 guidance largely unnecessary。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

这还有一个很实际的好处：

CFG 通常需要

[  
\text{conditional forward}  
+  
\text{unconditional forward}.  
]

如果只有中间一部分 timestep 开 CFG，那么其余 timestep 只需要一次 forward。

所以同时：

[  
\boxed{\text{quality ↑ + diversity ↑ + inference cost ↓}}  
]

而不是传统意义上的 quality-speed trade-off。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

---

# 4. 论文真正提出的方法有多复杂？

几乎没有复杂度。

本质就是给 CFG 加两个 hyperparameter：

[  
\boxed{\sigma_{\min},\sigma_{\max}}  
]

然后

[  
\text{CFG enabled}  
\iff  
\sigma\in[\sigma_{\min},\sigma_{\max}].  
]

所以它不是：

- 新 network；
    
- 新 loss；
    
- retraining；
    
- learnable guidance network；
    
- complicated adaptive scheduler。
    

而只是一个 **training-free sampling modification**。

也正因为这么简单，他们最后建议：

> 所有使用 guidance 的 diffusion model 都应该把 guidance interval 暴露成一个 sampling hyperparameter。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

---

# 5. 效果有多明显？

论文最抢眼的结果是在 **ImageNet 512×512**。

原本的 record FID：

[  
\mathrm{FID}=1.81  
]

加入 guidance interval 后：

[  
\boxed{\mathrm{FID}=1.40}  
]

而且不是换模型、增加训练数据或者大规模重新训练，仅仅改变 inference 时 guidance 的时间范围。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

作者还测试了不同：

- sampler parameters；
    
- network architectures；
    
- datasets；
    
- large-scale text-to-image setting；
    

包括 **Stable Diffusion XL**，都观察到了定量或定性的提升。([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

所以论文的 claim 并不是：

> “我们为 EDM 找到了一个特殊 trick。”

而更接近：

[  
\boxed{\text{Guidance 本质上应该是 time/noise dependent 的。}}  
]

---

# 6. 我觉得这篇论文最重要的地方

如果把它抽象一层，它其实是在挑战 CFG 中的一个隐藏假设：

传统方法默认

[  
\boxed{\text{conditioning importance is constant over diffusion time}}  
]

但实际上并不是。

不同时间段负责形成不同尺度的信息：

[  
\text{high noise}  
\rightarrow  
\text{global / coarse decisions}  
]

[  
\text{middle noise}  
\rightarrow  
\text{semantic structure}  
]

[  
\text{low noise}  
\rightarrow  
\text{local / fine detail}.  
]

而 conditional guidance 主要是一个 **semantic steering force**。

所以真正合理的是：

# [  
g(x_t,t)

w(t)  
\left[  
s_c(x_t,t)-s_u(x_t,t)  
\right],  
]

而不是

[  
w(t)=\text{constant}.  
]

这篇论文最简单地把 (w(t)) 设成一个 rectangular window：

[  
w(t)=w,\mathbf 1[t\in(t_1,t_2)].  
]

从这个角度看，**Guidance Interval 可以看成最简单的 time-dependent guidance scheduler**。

---

## 7. 对 diffusion / flow 来说，更值得思考的一点

如果站在 ODE/vector field 的角度看，CFG 相当于给原始 conditional vector field 加一个 correction：

# [  
v_{\text{guided}}(x,t)

v_c(x,t)  
+  
w(t)  
\underbrace{  
\left(v_c(x,t)-v_u(x,t)\right)  
}_{\Delta v(x,t)}.  
]

那么这篇论文的 empirical finding 就可以重新表述为：

[  
\boxed{  
\Delta v(x,t)\text{ 的有效性高度依赖于 }t  
}  
]

甚至更进一步：

**不是 correction 越久越好，而是 correction 应该只作用在 trajectory 最“可操纵”的时间窗口。**

这个观点后来确实成为很多 dynamic guidance / adaptive guidance 工作继续研究的方向；后续研究也把这类方法概括为 time-varying guidance，并试图解释为什么中间存在一个特别敏感的 guidance window。([arXiv](https://arxiv.org/html/2506.06085v2?utm_source=chatgpt.com "Feedback Guidance of Diffusion Models"))

所以如果只用一句话记住这篇论文：

[  
\boxed{  
\textbf{Don't guide harder; guide at the right time.}  
}  
]

或者更具体：

> **CFG 的主要问题之一不是 scale，而是 timing。High-noise 时开 guidance 会损失 distribution coverage，low-noise 时开 guidance 又浪费计算；只在中间 noise interval 开启，反而同时改善 FID、diversity 和速度。** ([arXiv](https://arxiv.org/abs/2404.07724 "[2404.07724] Applying Guidance in a Limited Interval Improves Sample and Distribution Quality in Diffusion Models"))

如果你在研究 diffusion / flow，我觉得这篇 paper 更值得继续追的是：**为什么“中间时间窗口”是最有效的？能不能从 score / posterior / flow trajectory 的角度把这个 interval 推出来，而不是 grid search (\sigma_{\min},\sigma_{\max})**。这会比“把 CFG 改成一个 window”本身更有研究味道。
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
> `ingest raw/literature-notes/kynkaanniemiApplyingGuidanceLimited2024.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/kynkaanniemiApplyingGuidanceLimited2024.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-11T17:14:19.941+08:00 %%
