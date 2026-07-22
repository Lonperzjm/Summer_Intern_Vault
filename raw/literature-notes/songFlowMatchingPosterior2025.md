---
type: literature-note
citekey: songFlowMatchingPosterior2025
title: "Flow Matching Posterior Sampling: A Training-free Conditional Generation for Flow Matching"
aliases: ["@songFlowMatchingPosterior2025"]
authors: "Kaiyu Song, Hanjiang Lai, Yan Pan, Kun Yue, Jian yin"
firstAuthor: "Song"
year: 2025
itemType: preprint
doi: "10.48550/arXiv.2411.07625"
url: "http://arxiv.org/abs/2411.07625"
zotero: "zotero://select/library/items/T4NLVQS6"
tags: [literature, computer-science---computer-vision-and-pattern-recognition]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-06-29
updated: 2026-06-29
ingested_to_wiki: true  # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/songFlowMatchingPosterior2025]]"
---

# Flow Matching Posterior Sampling: A Training-free Conditional Generation for Flow Matching

> [!info] @songFlowMatchingPosterior2025 · Song et al. · 2025
> [Open in Zotero](zotero://select/library/items/T4NLVQS6) · [DOI](https://doi.org/10.48550/arXiv.2411.07625) · [URL](http://arxiv.org/abs/2411.07625) · [PDF](file:///home/lonper/Zotero/storage/VHTCYJCK/Song%20等%20-%202025%20-%20Flow%20Matching%20Posterior%20Sampling%20A%20Training-free%20Conditional%20Generation%20for%20Flow%20Matching.pdf)

## Abstract

> [!abstract]- Click to expand
> Training-free conditional generation based on flow matching aims to leverage pre-trained unconditional flow matching models to perform conditional generation without retraining. Recently, a successful training-free conditional generation approach incorporates conditions via posterior sampling, which relies on the availability of a score function in the unconditional diffusion model. However, flow matching models do not possess an explicit score function, rendering such a strategy inapplicable. Approximate posterior sampling for flow matching has been explored, but it is limited to linear inverse problems. In this paper, we propose Flow Matching-based Posterior Sampling (FMPS) to expand its application scope. We introduce a correction term by steering the velocity field. This correction term can be reformulated to incorporate a surrogate score function, thereby bridging the gap between flow matching models and score-based posterior sampling. Hence, FMPS enables the posterior sampling to be adjusted within the flow matching framework. Further, we propose two practical implementations of the correction mechanism: one aimed at improving generation quality, and the other focused on computational efficiency. Experimental results on diverse conditional generation tasks demonstrate that our method achieves superior generation quality compared to existing state-of-the-art approaches, validating the effectiveness and generality of FMPS.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 来看看已有的方法
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

### Imported 2026-06-29 18:10

- 🟡 **p.3** — MPGD [13] introduced the linear manifold theory to avoid gradient calculation, thereby alleviating the computation cost. [⤴](zotero://open-pdf/library/items/VHTCYJCK?page=3&annotation=7W4DIA6H)

- 🟡 **p.5** — vθ(xt, t) =   ̇bt  bt xt − at(  ̇at −   ̇bt  bt at)∇xt log p(xt), [⤴](zotero://open-pdf/library/items/VHTCYJCK?page=5&annotation=FDQN5N5N)
  - 💬 *我的批注*：a,b 反了

- 🟡 **p.7** — xˆ0|t = xt − tvθ(xt, t). (25) [⤴](zotero://open-pdf/library/items/VHTCYJCK?page=7&annotation=NH6QZPH4)

- 🟡 **p.7** — x0 = 1  at  (xt − btx1). [⤴](zotero://open-pdf/library/items/VHTCYJCK?page=7&annotation=WG4T2VRI)

- 🟡 **p.7** — g1 = ||v(xt, t)||2 [⤴](zotero://open-pdf/library/items/VHTCYJCK?page=7&annotation=FVAT4SPL)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
# FMPS 方法主线

Flow Matching 前向路径：

$$
x_t = a_t x_0 + b_t \epsilon.
$$

Proposition 1 把速度场改写成 score 形式：

$$
v_\theta(x_t,t)
=
\frac{\dot b_t}{b_t}x_t
+
a_t
\left(
\dot a_t - \frac{\dot b_t}{b_t}a_t
\right)
\nabla_{x_t}\log p_t(x_t).
$$

令：

$$
\beta_t
=
a_t
\left(
\dot a_t - \frac{\dot b_t}{b_t}a_t
\right).
$$

通过贝叶斯公式：

$$
\nabla_{x_t}\log p_t(x_t \mid c)
=
\nabla_{x_t}\log p_t(x_t)
+
\nabla_{x_t}\log p(c \mid x_t).
$$

所以条件速度场为：

$$
v_{\text{guided}}
=
v_\theta
+
r\beta_t\nabla_{x_t}\log p(c \mid x_t).
$$

再用 FreeDoM 式距离近似：

$$
\nabla_{x_t}\log p(c \mid x_t)
\approx
-\nabla_{x_t}D(\hat{x}_{0|t},c).
$$

最终得到：

$$
\boxed{
v_{\text{guided}}
=
v_\theta
-
r\beta_t\nabla_{x_t}D(\hat{x}_{0|t},c).
}
$$

---

# 两种 $\hat{x}_{0|t}$ 估计方式

## 1. FMPS-gradient

一步 Euler 估计：

$$
\hat{x}_{0|t}
=
x_t
-
t v_\theta(x_t,t).
$$

此时：

$$
\nabla_{x_t}D(\hat{x}_{0|t},c)
=
\left(
I
-
t
\frac{\partial v_\theta(x_t,t)}{\partial x_t}
\right)^\top
\nabla_{\hat{x}_{0|t}}D.
$$

优点是梯度更准；缺点是要反传 through $v_\theta$，计算贵。

---

## 2. FMPS-free

用前向路径反解：

$$
\hat{x}_{0|t}
=
\frac{x_t - b_t x_1}{a_t}.
$$

此时：

$$
\nabla_{x_t}D(\hat{x}_{0|t},c)
=
\frac{1}{a_t}
\nabla_{\hat{x}_{0|t}}D.
$$

优点是不用算

$$
\frac{\partial v_\theta}{\partial x_t},
$$

计算便宜；缺点是估计更粗，且 $t=1$ 时 $a_t=0$，第一步不能用。

---

# 最后实际算法常用归一化

$$
g^1
=
\frac{
\|v_\theta(x_t,t)\|_2
}{
\|\nabla_{x_t}D(\hat{x}_{0|t},c)\|_2
}
\nabla_{x_t}D(\hat{x}_{0|t},c).
$$

最终采样速度场：

$$
\boxed{
v_{\text{guided}}
=
v_\theta
-
r\beta_t g^1.
}
$$

一句话：

$$
\boxed{
x_t
\to
\hat{x}_{0|t}
\to
D(\hat{x}_{0|t},c)
\to
\nabla_{x_t}D
\to
v_\theta - r\beta_t g^1.
}
$$
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/training-free-guidance]]、[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/tweedie-formula]]、[[wiki/concepts/energy-guidance]]
- 方法：[[wiki/methods/fmps]]、[[wiki/methods/freedom]]、[[wiki/methods/dps]]
- 实体（作者 / 模型 / 机构）：Kaiyu Song、Hanjiang Lai、Yan Pan、Jian Yin（中山大学 SYSU）
- 基线 / 对比：FreeDoM（diffusion 版同款）；MPGD（省算力对照）；TFG-Flow
- 本篇 wiki source 页：[[wiki/sources/songFlowMatchingPosterior2025]]（占掉 energy-guidance 候选①轴，见 [[wiki/synthesis/energy-guidance-landscape]]）
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
> `ingest raw/literature-notes/songFlowMatchingPosterior2025.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/songFlowMatchingPosterior2025.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-06-29T18:10:12.664+08:00 %%
