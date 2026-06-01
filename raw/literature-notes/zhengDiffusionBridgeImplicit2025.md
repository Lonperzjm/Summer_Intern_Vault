---
type: literature-note
citekey: zhengDiffusionBridgeImplicit2025
title: "Diffusion Bridge Implicit Models"
aliases: ["@zhengDiffusionBridgeImplicit2025"]
authors: "Kaiwen Zheng, Guande He, Jianfei Chen, Fan Bao, Jun Zhu"
firstAuthor: "Zheng"
year: 2025
itemType: preprint
doi: "10.48550/arXiv.2405.15885"
url: "http://arxiv.org/abs/2405.15885"
zotero: "zotero://select/library/items/THAV4DG6"
tags: [literature, todo]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-06-01
updated: 2026-06-01
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/zhengDiffusionBridgeImplicit2025]]"
---

# Diffusion Bridge Implicit Models

> [!info] @zhengDiffusionBridgeImplicit2025 · Zheng et al. · 2025
> [Open in Zotero](zotero://select/library/items/THAV4DG6) · [DOI](https://doi.org/10.48550/arXiv.2405.15885) · [URL](http://arxiv.org/abs/2405.15885) · [PDF](file:///home/lonper/Zotero/storage/Y3M3Y5BV/Zheng%20等%20-%202025%20-%20Diffusion%20Bridge%20Implicit%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Denoising diffusion bridge models (DDBMs) are a powerful variant of diffusion models for interpolating between two arbitrary paired distributions given as endpoints. Despite their promising performance in tasks like image translation, DDBMs require a computationally intensive sampling process that involves the simulation of a (stochastic) differential equation through hundreds of network evaluations. In this work, we take the first step in fast sampling of DDBMs without extra training, motivated by the well-established recipes in diffusion models. We generalize DDBMs via a class of non-Markovian diffusion bridges defined on the discretized timesteps concerning sampling, which share the same marginal distributions and training objectives, give rise to generative processes ranging from stochastic to deterministic, and result in diffusion bridge implicit models (DBIMs). DBIMs are not only up to 25$\times$ faster than the vanilla sampler of DDBMs but also induce a novel, simple, and insightful form of ordinary differential equation (ODE) which inspires high-order numerical solvers. Moreover, DBIMs maintain the generation diversity in a distinguished way, by using a booting noise in the initial sampling step, which enables faithful encoding, reconstruction, and semantic interpolation in image translation tasks. Code is available at https://github.com/thu-ml/DiffusionBridge.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 大概就是，把$p(x_n|x_{n+1})$换成$p(x_n|x_0)$，然后换成$p(x_n|x_{n+1},x_0)$，然后得到大幅推进公式
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
### Imported 2026-06-01 21:19

- 🟡 **p.2** — DBIMs can induce a novel form of ordinary differential equation (ODE), [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=2&annotation=EBQY2PQ3)

- 🟡 **p.3** — starting from xT = y:  dxt = f (t)xt − g2(t) ∇xt log q(xt|xT = y) − ∇xt log qT |t(xT = y|xt) dt + g(t)dw ̄t, (7) dxt = f (t)xt − g2(t) 1  2 ∇xt log q(xt|xT = y) − ∇xt log qT |t(xT = y|xt) dt. (8) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=3&annotation=QGMLIRG6)

- 🟡 **p.3** — lacks theoretical insights in developing efficient diffusion samplers. [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=3&annotation=UZ9KRVVJ)

- 🟡 **p.4** — as long as they agree on the N marginals {q(xtn |xT )}N−1  n=0 [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=NIV3RDYB)

- 🟡 **p.4** — q(ρ)(xtn |x0, xtn+1 , xT ) = N (atn xT + btn x0 +  q  ct2n − ρ2n  xtn+1 − atn+1 xT − btn+1 x0  ctn+1  , ρ2  nI) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=2NZEEIIG)

- 🟡 **p.4** — Proposition 3.1 (Marginal Preservation, proof in Appendix B.1). For 0 ≤ n ≤ N − 1, we have q(ρ)(xtn |xT ) = q(xtn |xT ). [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=WSSKIU97)
  - 💬 *我的批注*：我们能证明更强的 endpoint-conditioned 版本：  
        
      $$  
      q^{(p)}(x_{t_n}\mid x_0,x_T)  
      =  
      q(x_{t_n}\mid x_0,x_T).  
      $$  
        
      这导致这个sampling过程可以使用ddpm的score

- 🟡 **p.4** — q(ρ)(xtn |xθ(xtn+1 , tn+1, xT ), xtn+1 , xT ), 1 ≤ n ≤ N − 1 (12) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=GTS7ZWML)

- 🟡 **p.4** — J (ρ)(θ) = Eq(xT )Eq(ρ)(xt0:N−1 |xT )  h  log q(ρ)(xt1:N−1 |x0, xT ) − log pθ(xt0:N−1 |xT )  i  (13) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=QS6P7IZU)

- 🟡 **p.4** — equivalent [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=4&annotation=ESBG8YS6)

- 🟡 **p.5** — sθ(xt, t, xT ) = − xt − atxT − btxθ(xt, t, xT ) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=5&annotation=IF64MHLU)

- 🟡 **p.5** — xtn = atn xT + btn xˆ0 +  q  ct2n − ρ2n  xtn+1 − atn+1 xT − btn+1 xˆ0  ctn+1  | {z }  predicted noise εˆ  +ρnε, ε ∼ N (0, I) (15) [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=5&annotation=C2TKAXC3)

- 🟡 **p.5** — ρn = 0 for each 0 ≤ n ≤ N −1, the inference process will be free from random noise and composed of deterministic iterative updates, characteristic of an implicit probabilistic model [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=5&annotation=QBDKVH4L)

- 🟡 **p.6** — The Singularity at the Initial Step for Deterministic Sampling [⤴](zotero://open-pdf/library/items/Y3M3Y5BV?page=6&annotation=ZCPELPNU)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. DBIM 的核心就是 DDBM 的 DDIM 版本：保持 DDBM 的 bridge marginal
	$$
	q(x_t\mid x_0,x_T)
	=
	\mathcal{N}
	\left(
	a_t x_T + b_t x_0,
	c_t^2 I
	\right)
	$$	
	不变，但改变 bridge joint / transition。这样原来的 DDBM bridge score
	$$
	s_\theta(x_t,t,x_T)
	\approx
	\nabla_{x_t}\log q(x_t\mid x_T)
	$$
	可以直接复用，所以 DBIM 不需要额外训练。

2. DBIM 的采样公式本质是把真实 $x_0$ 换成预测的 $\hat{x}_0$，再从上一时刻反推出 predicted bridge noise：
	$$
	\hat{\epsilon}
	=
	\frac{
	x_{t_{n+1}}
	-
	a_{t_{n+1}}x_T
	-
	b_{t_{n+1}}\hat{x}_0
	}{
	c_{t_{n+1}}
	}.
	$$
	然后更新为
	$$
	x_{t_n}
	=
	a_{t_n}x_T
	+
	b_{t_n}\hat{x}_0
	+
	\sqrt{
	c_{t_n}^2-\rho_n^2
	}\,\hat{\epsilon}
	+
	\rho_n\epsilon.
	$$
	这和 DDIM 的“预测 $x_0$ + 继承噪声 + 可选新噪声”完全对应，只是普通 diffusion 的 $\alpha_t,\sigma_t$ 被 bridge 的 $a_t,b_t,c_t$ 替代。

3. $\rho_n$ 是 DBIM 的随机性控制参数，对应 DDIM 里的 $\eta/\sigma_t$。$\rho_n=0$ 时后续采样是 deterministic implicit bridge；$\rho_n>0$ 时有随机性。初始步因为 $c_T=0$ 会奇异，所以必须引入 booting noise。这个 booting noise 在固定 $x_T$ 下相当于 latent variable，使 DBIM 可以做 faithful reconstruction、encoding 和 semantic interpolation。
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
> `ingest raw/literature-notes/zhengDiffusionBridgeImplicit2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/zhengDiffusionBridgeImplicit2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-01T21:19:18.743+08:00 %%
