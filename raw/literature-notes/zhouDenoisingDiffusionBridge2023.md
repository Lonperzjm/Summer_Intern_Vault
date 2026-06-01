---
type: literature-note
citekey: zhouDenoisingDiffusionBridge2023
title: Denoising Diffusion Bridge Models
aliases:
  - "@zhouDenoisingDiffusionBridge2023"
authors: Linqi Zhou, Aaron Lou, Samar Khanna, Stefano Ermon
firstAuthor: Zhou
year: 2023
itemType: preprint
doi: 10.48550/arXiv.2309.16948
url: http://arxiv.org/abs/2309.16948
zotero: zotero://select/library/items/VQQALXDY
tags:
  - literature
  - important
status: read
priority: P1
my-rating: "3.8"
created: 2026-06-01
updated: 2026-06-01
ingested_to_wiki: true
wiki_page: "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]"
---

# Denoising Diffusion Bridge Models

> [!info] @zhouDenoisingDiffusionBridge2023 · Zhou et al. · 2023
> [Open in Zotero](zotero://select/library/items/VQQALXDY) · [DOI](https://doi.org/10.48550/arXiv.2309.16948) · [URL](http://arxiv.org/abs/2309.16948) · [PDF](file:///home/lonper/Zotero/storage/W84Y2DB9/Zhou%20等%20-%202023%20-%20Denoising%20Diffusion%20Bridge%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Diffusion models are powerful generative models that map noise to data using stochastic processes. However, for many applications such as image editing, the model input comes from a distribution that is not random noise. As such, diffusion models must rely on cumbersome methods like guidance or projected sampling to incorporate this information in the generative process. In our work, we propose Denoising Diffusion Bridge Models (DDBMs), a natural alternative to this paradigm based on diffusion bridges, a family of processes that interpolate between two paired distributions given as endpoints. Our method learns the score of the diffusion bridge from data and maps from one endpoint distribution to the other by solving a (stochastic) differential equation based on the learned score. Our method naturally unifies several classes of generative models, such as score-based diffusion models and OT-Flow-Matching, allowing us to adapt existing design and architectural choices to our more general problem. Empirically, we apply DDBMs to challenging image datasets in both pixel and latent space. On standard image translation problems, DDBMs achieve significant improvement over baseline methods, and, when we reduce the problem to image generation by setting the source distribution to random noise, DDBMs achieve comparable FID scores to state-of-the-art methods despite being built for a more general task.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 我要向你正式的推荐生成元法：[[生成元方法对于SDE]].
- 数学嘉豪，probability jargon imposter，math class clown，玩弄数学的家伙。
- 但是方法好像部分可行？可以改进为score-based bridge sde啥的。
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
### Imported 2026-06-01 10:04

- 🟡 **p.1** — Our method learns the score of the diffusion bridge from data and maps from one endpoint distribution to the other by solving a (stochastic) differential equation based on the learned score. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=1&annotation=NIP2JS3S)

- 🟡 **p.1** — unifies several classes of generative models, [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=1&annotation=25Z8M4N6)

- 🟡 **p.1** — which directly model a transport between two arbitrary probability distributions [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=1&annotation=J4VEB2QM)

- ⚪ **p.1** — For instance, ODE based flow-matching methods (Lipman et al., 2023; Albergo and Vanden-Eijnden, 2023; Liu et al., 2022a), which learn a deterministic path between two arbitrary probability distributions, have mainly been applied to image generation problems and have not been investigated for image translation. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=1&annotation=TJLYFD4D)

- 🔴 **p.1** — Furthermore, on image generation, ODE methods have not achieved the same empirical success as diffusion models. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=1&annotation=D6L52YJ3)
  - 💬 *我的批注*：现在不一定成立

- 🟡 **p.2** — use this perspective to establish a general framework for distribution translation. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=2&annotation=FP9WF9B5)

- 🟡 **p.2** — WITH FIXED ENDPOINTS [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=2&annotation=QLZ4FXXI)

- 🟡 **p.2** — condition a diffusion process on a fixed known endpoint via the famous Doob’s h-transform [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=2&annotation=QZHHUBLL)

- 🟡 **p.3** — dxt = f (xt, t)dt + g(t)2h(xt, t, y, T ) + g(t)dwt, x0 ∼ qdata(x), xT = y (5) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=3&annotation=LQP5TVHS)
  - 💬 *我的批注*：推导[[raw/notes/生成元方法对于SDE]]

- 🟡 **p.3** — qdata(x | y) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=3&annotation=DP9LD65F)

- 🟡 **p.3** — We can construct the time-reversed SDE/probability flow ODE of q(xt | xT ) via the following theorem. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=3&annotation=PBTSDW7E)

- 🟡 **p.3** — Theorem 1. The evolution of conditional probability q(xt | xT ) has a time-reversed SDE of the form  dxt =  h  f (xt, t) − g2(t) s(xt, t, y, T ) − h(xt, t, y, T )  i  dt + g(t)dwˆ t, xT = y (6) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=3&annotation=BL5QJ4QU)

- 🟡 **p.3** — dxt =  h  f (xt, t) − g2(t) 1  2 s(xt, t, y, T ) − h(xt, t, y, T )  i  dt, xT = y (7) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=3&annotation=PUXGU46J)

- 🟡 **p.4** — s(x, t, y, T ) = ∇xt log q(xt | xT ) xt=x,xT =y [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=AHXG9Z94)
  - 💬 *我的批注*：唯一不知道要学的

- 🟡 **p.4** — s(x, t, y, T ) = ∇xt log q(xt | xT ) xt=x,xT =y where q(xt | xT ) = R  x0 q(xt | x0, xT )qdata(x0 | xT )dx0. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=BGRT7PN5)
  - 💬 *我的批注*：也就是：  
        
      $$  
      \nabla_{x_t}\log q(x_t\mid x_T)  
      =  
      \mathbb{E}_{x_0\mid x_t,x_T}  
      \left[  
      \nabla_{x_t}\log q(x_t\mid x_0,x_T)  
      \right].  
      $$

- 🟡 **p.4** — and (x0, xT ) in our case) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=85V46FRC)

- 🟡 **p.4** — q(xt | x0, xT ) = N (μˆt, σˆ2  t I), where  μˆt = SNRT  SNRt  αt αT  xT + αtx0(1 − SNRT  SNRt  )  σˆ2  t = σ2  t (1 − SNRT  SNRt  ) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=F5UPK3DM)
  - 💬 *我的批注*：bayes得到

- ⚫ **p.4** — SNRt = αt2/σt2 is the signal-tonoise ratio at time t. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=Z26DM5HN)

- 🟡 **p.4** — we present the bridge processes generated by both VP and VE diffusion in Table 1 and recommend choosing f and g specified therein. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=4&annotation=TDZHCX5F)
  - 💬 *我的批注*：## 第一层：选普通扩散过程  
        
      选择 VP 或 VE，指定：  
        
      $$  
      p(\mathbf{x}_t\mid \mathbf{x}_0)  
      =  
      \mathcal{N}  
      \left(  
      \alpha_t\mathbf{x}_0,  
      \sigma_t^2 I  
      \right)  
      $$  
        
      或  
        
      $$  
      p(\mathbf{x}_t\mid \mathbf{x}_0)  
      =  
      \mathcal{N}  
      \left(  
      \mathbf{x}_0,  
      \sigma_t^2 I  
      \right).  
      $$  
        
      由此得到：  
        
      $$  
      \mathbf{f},  
      \qquad  
      g^2(t).  
      $$  
        
      ## 第二层：得到未来转移核  
        
      由线性高斯性质得到：  
        
      $$  
      p(\mathbf{x}_T\mid \mathbf{x}_t).  
      $$  
        
      ---  
        
      ## 第三层：得到 Doob 引导项  
        
      计算：  
        
      $$  
      \mathbf{h}  
      =  
      \nabla_{\mathbf{x}_t}  
      \log p(\mathbf{x}_T\mid \mathbf{x}_t).  
      $$  
        
      这给出表 1 最后一列。  
        
      ---  
        
      ## 第四层：定义 DDBM 的 bridge 采样分布  
        
      规定：  
        
      $$  
      q(\mathbf{x}_t\mid \mathbf{x}_0,\mathbf{x}_T)  
      :=  
      p(\mathbf{x}_t\mid \mathbf{x}_0,\mathbf{x}_T).  
      $$  
        
      再用贝叶斯公式：  
        
      $$  
      p(\mathbf{x}_t\mid \mathbf{x}_0,\mathbf{x}_T)  
      \propto  
      p(\mathbf{x}_T\mid \mathbf{x}_t)\,  
      p(\mathbf{x}_t\mid \mathbf{x}_0).  
      $$  
        
      因为两项都是高斯，所以得到公式 $(8)$ 的高斯分布。

- 🟡 **p.5** — L(θ) = Ext,x0,xT ,t  h  w(t)∥sθ(xt, xT , t) − ∇xt log q(xt | x0, xT )∥2i  (9)  satisfies sθ(xt, xT , t) = ∇xt log q(xt | xT ). [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=5&annotation=2B9PIEIJ)

- 🟡 **p.5** — PARAMETERIZATION [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=5&annotation=ECQ9ZK3C)

- 🟡 **p.5** — Score reparameterization. [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=5&annotation=XTNPQN3B)

- 🟡 **p.5** — ∇xt log q(xt | xT ) ≈ −  xt − SNRT  SNRt  αt αT xT + αtDθ(xt, xT , t)(1 − SNRT  SNRt )  σt2(1 − SNRT  SNRt ) (10) [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=5&annotation=L9FRURU8)

- 🟡 **p.5** — dxt =  h  f (xt, t) − g2(t) 1  2 s(xt, t, y, T ) − wh(xt, t, y, T )  i  dt, xT = y [⤴](zotero://open-pdf/library/items/W84Y2DB9?page=5&annotation=PV3L8N52)
  - 💬 *我的批注*：h 在正向 bridge 中拉向 y；但在从 y 到 x0 的反向采样中，wh 项实际推动样本离开 y。

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 是这样，本质上我们是找一个x->y的路径，也就是在两个端点之间建立 bridge：
	$$
	\mathbf{x}_0
	\to
	\mathbf{x}_t
	\to
	\mathbf{x}_T.
	$$
	 “stochastic bridge” 说明中间路径不是确定性的直线，而是随机扩散桥。对于 VE / VP 这类高斯转移核，中间点可写成类似：
	$$
	\mathbf{x}_t
	=
	a_t\mathbf{x}_T
	+
	b_t\mathbf{x}_0
	+
	\sqrt{c_t}\epsilon,
	\qquad
	\epsilon\sim\mathcal{N}(0,I).
	$$
	这样的bridge分布有很多种走法，通过由conditional diffusion而来的概率公式，推导出如下游走方法![[zhouDenoisingDiffusionBridge2023-1780285042493.webp]]公式的解释很简单，就是$h$是保证能到达$x_T$的流，$s$是包含概率分布，保证逆反扩散过程的流
2.  基本的物理图像是，一个$x_T$对应的一个$$
	\mathbf{x}_0
	\to
	\mathbf{x}_t
	\to
	\mathbf{x}_T.
	$$流，或者说桥。$x_0$可以是一个分布，但是$x_T$通常是确定的。在这个一对多的流分布里，有：
	$$
	\mathcal{L}(\theta)
	=
	\mathbb{E}_{\mathbf{x}_t,\mathbf{x}_0,\mathbf{x}_T,t}
	\left[
	w(t)
	\left\|
	s_\theta(\mathbf{x}_t,\mathbf{x}_T,t)
	-
	\nabla_{\mathbf{x}_t}
	\log q(\mathbf{x}_t\mid \mathbf{x}_0,\mathbf{x}_T)
	\right\|^2
	\right]
	\tag{9}
	$$
	满足：
	$$
	s_\theta(\mathbf{x}_t,\mathbf{x}_T,t)
	=
	\nabla_{\mathbf{x}_t}
	\log q(\mathbf{x}_t\mid \mathbf{x}_T).
	$$
	而这个新分布满足边界分布：
	$$
	q_{\mathrm{data}}(\mathbf{x}|\mathbf{y}).
	$$
3. 然后我们寄希望于U-net能理解语义，对每个$y$构造出完美的$$
	q_{\mathrm{data}}(\mathbf{x}|\mathbf{y}).
	$$
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
> `ingest raw/literature-notes/zhouDenoisingDiffusionBridge2023.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/zhouDenoisingDiffusionBridge2023.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-01T10:04:41.167+08:00 %%
