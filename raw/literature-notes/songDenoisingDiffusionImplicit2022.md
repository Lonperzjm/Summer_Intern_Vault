---
type: literature-note
citekey: songDenoisingDiffusionImplicit2022
title: Denoising Diffusion Implicit Models
aliases:
  - "@songDenoisingDiffusionImplicit2022"
authors: Jiaming Song, Chenlin Meng, Stefano Ermon
firstAuthor: Song
year: 2022
itemType: preprint
doi: 10.48550/arXiv.2010.02502
url: http://arxiv.org/abs/2010.02502
zotero: zotero://select/library/items/E64K6V85
tags:
  - literature
  - todo
status: read
priority: P1
my-rating: "4"
created: 2026-05-14
updated: 2026-05-14
ingested_to_wiki: true
wiki_page: "[[wiki/sources/songDenoisingDiffusionImplicit2022]]"
---

# Denoising Diffusion Implicit Models

> [!info] @songDenoisingDiffusionImplicit2022 · Song et al. · 2022
> [Open in Zotero](zotero://select/library/items/E64K6V85) · [DOI](https://doi.org/10.48550/arXiv.2010.02502) · [URL](http://arxiv.org/abs/2010.02502) · [PDF](file:///home/lonper/Zotero/storage/LVNKQK7U/Song%20等%20-%202022%20-%20Denoising%20Diffusion%20Implicit%20Models.pdf)

## Abstract

> [!abstract]- Click to expand
> Denoising diffusion probabilistic models (DDPMs) have achieved high quality image generation without adversarial training, yet they require simulating a Markov chain for many steps to produce a sample. To accelerate sampling, we present denoising diffusion implicit models (DDIMs), a more efficient class of iterative implicit probabilistic models with the same training procedure as DDPMs. In DDPMs, the generative process is defined as the reverse of a Markovian diffusion process. We construct a class of non-Markovian diffusion processes that lead to the same training objective, but whose reverse process can be much faster to sample from. We empirically demonstrate that DDIMs can produce high quality samples $10 \times$ to $50 \times$ faster in terms of wall-clock time compared to DDPMs, allow us to trade off computation for sample quality, and can perform semantically meaningful image interpolation directly in the latent space.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- 学习sampling方法
- 更加直觉上了解duffusion方法
- 了解sampling的原因，了解flow
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
### Imported 2026-05-11 19:08

- 🟡 **p.2** — In Section 3, we generalize the forward diffusion process used by DDPMs, which is Markovian, to non-Markovian ones, for which we are still able to design suitable reverse generative Markov chains. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=2&annotation=ZHPKJUUA)

- 🟡 **p.2** — In particular, we are able to use non-Markovian diffusion processes which lead to ”short” generative Markov chains (Section 4.2) that can be simulated in a small number of steps. This can massively increase sample efficiency only at a minor cost in sample quality. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=2&annotation=FYDNDHDG)

### Imported 2026-05-14 17:30

- 🟣 **p.1** — 1 INTRODUCTION [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=1&annotation=9K9D6QRI)
  - 💬 *我的批注*：DDIM 保持 DDPM 的训练目标不变，但把正向扩散过程从马尔可夫过程推广为非马尔可夫过程。这样，同一个训练好的噪声预测网络可以对应一大族不同的生成过程。通过选择合适的非马尔可夫过程，DDIM 可以构造更短、更快的反向生成链，在采样速度大幅提升的同时保持较好的图像质量，并带来 DDPM 不具备的 consistency 和 latent-space semantic interpolation 能力。

- 🔴 **p.3** — However, as all T iterations have to be performed sequentially, instead of in parallel, to obtain a sample x0, sampling from DDPMs is much slower than sampling from other deep generative models, which makes them impractical for tasks where compute is limited and latency is critical. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=3&annotation=P9SHSZA6)

- 🟡 **p.3** — These non-Markovian inference process lead to the same surrogate objective function as DDPM [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=3&annotation=BXI257ZU)

- ⚪ **p.3** — In Appendix A, we show that the non-Markovian perspective also applies beyond the Gaussian case. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=3&annotation=YZ4I42PS)
  - 💬 *我的批注*：wc到时候看看咋回事

- 🟢 **p.3** — The mean function is chosen to order to ensure that qσ(xt|x0) = N (√αtx0, (1 − αt)I) for all t [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=3&annotation=WNDV4VR8)

- 🟡 **p.4** — p(t)  θ (xt−1|xt) =  {  N (f (1)  θ (x1), σ12I) if t = 1  qσ(xt−1|xt, f (t)  θ (xt)) otherwise, (10) [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=4&annotation=YAPDDAWP)

- 🟡 **p.4** — Theorem 1. For all σ > 0, there exists γ ∈ RT>0 and C ∈ R, such that Jσ = Lγ + C. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=4&annotation=JRUGQ2HB)

- 🟡 **p.4** — Therefore, if parameters are not shared across t in the model θ, then the L1 objective used by Ho et al. (2020) can be used as a surrogate objective for the variational objective Jσ as well. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=4&annotation=XA7RRUKE)
  - 💬 *我的批注*：以后假设都独立，反正能成

- 🟡 **p.5** — 4.2 ACCELERATED GENERATION PROCESSES [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=5&annotation=QKL725C3)

- 🟡 **p.5** — We show that only slight changes to the updates in Eq. (12) are needed to obtain the new, faster generative processes, which applies to DDPM, DDIM, as well as all generative processes considered in Eq. (10). [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=5&annotation=HM7V4QVL)
  - 💬 *我的批注*：甚至可以直接跳到t=0,因为我们的正向过程是不加修饰的，也就是正向噪声与t无关。

- 🟡 **p.6** — dx ̄(t) = (t)  θ  ( x ̄(t)  √σ2 + 1  )  dσ(t), [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=6&annotation=EV7NPYKP)

- 🔴 **p.6** — This suggests that with enough discretization steps, the we can also reverse the generation process (going from t = 0 to T ), which encodes x0 to xT and simulates the reverse of the ODE in Eq. (14). [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=6&annotation=BPPFWCUY)

- 🔴 **p.7** — As expected, the sample quality becomes higher as we increase dim(τ ), presenting a tradeoff between sample quality and computational costs. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=7&annotation=KQYKQDHC)

- 🟡 **p.7** — Notably, DDIM is able to produce samples with quality comparable to 1000 step models within 20 to 100 steps, which is a 10× to 50× speed up compared to the original DDPM. [⤴](zotero://open-pdf/library/items/LVNKQK7U?page=7&annotation=WA9SVQ7U)

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 因为正向噪声与t无关，并且训练时噪声也是t-0的噪声，所以去噪过程中可以跳步
2. 或许可以改进加噪过程与正确噪声的选取？只要符合分步且全局的思想。比如说t：0->10->t，正确噪声选第二段？然后设计的“->”次数最大为sampling设计次数，比如20次。
3.  $d\bar{x}(t)=\epsilon_{\theta}^{(t)}\left(\frac{\bar{x}(t)}{\sqrt{\sigma^2+1}}\right)d\sigma(t)$ 揭示了为什么tdim大效果好，因为不应该期望画家能一次性完成包括从大纲到作画细节上的所有工作
4. 惊人的，$t_0$的图像框架与$T$ 纯噪声有关 
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/non-markovian-diffusion]]（本文核心构造）、[[wiki/concepts/epsilon-parameterization]]（能复用 DDPM 网络的根因）、[[wiki/concepts/diffusion-process]]（松绑了"前向必须马尔可夫"）、[[wiki/concepts/score-sde]]（ODE 视角 = probability-flow ODE）
- 方法：[[wiki/methods/ddim]]（本文奠基）、[[wiki/methods/ddpm]]（共享训练目标与 ε 网络）
- 实体：[[wiki/entities/jiaming-song]]、[[wiki/entities/stefano-ermon]]、[[wiki/entities/stanford]]
- 基线 / 对比：vs DDPM —— 同质量下快 10–50×；σ 从 0 到 σ_DDPM 是一条连续谱（σ=0 即 DDIM，σ=σ_DDPM 即退回 DDPM）
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
- 验证 overview 推论 3「采样速度是开放赛道」：加速可以是纯采样期的事，不重训、不改 backbone —— 对要反复采样的编辑场景尤其关键
- σ=0 的确定性 + ODE 可反向积分 = DDIM inversion 的雏形，是 inversion-based 编辑那一派的理论入口
- 强化可变性光谱：DDIM 只动"采样链的形状（步数、σ）"，属于"研究杠杆"那一档且代价极低 —— "范式不变、组件可调"的又一个干净样本
- 我总结第 2 条那个 idea（分段 / 自适应选噪声）其实落在非马尔可夫族的延长线上，可探索点：可学习或数据自适应的子序列 τ
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
> `ingest raw/literature-notes/songDenoisingDiffusionImplicit2022.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/songDenoisingDiffusionImplicit2022.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-14T17:30:35.728+08:00 %%
