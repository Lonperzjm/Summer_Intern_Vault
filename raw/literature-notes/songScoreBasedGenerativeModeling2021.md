---
type: literature-note
citekey: songScoreBasedGenerativeModeling2021
title: Score-Based Generative Modeling through Stochastic Differential Equations
aliases:
  - "@songScoreBasedGenerativeModeling2021"
authors: Yang Song, Jascha Sohl-Dickstein, Diederik P. Kingma, Abhishek Kumar, Stefano Ermon, Ben Poole
firstAuthor: Song
year: 2021
itemType: preprint
doi: 10.48550/arXiv.2011.13456
url: http://arxiv.org/abs/2011.13456
zotero: zotero://select/library/items/FUTSN73A
tags:
  - literature
  - todo
status: read
priority: P1
my-rating: "5"
created: 2026-05-20
updated: 2026-05-20
ingested_to_wiki: true
wiki_page: "[[wiki/sources/songScoreBasedGenerativeModeling2021]]"
---

# Score-Based Generative Modeling through Stochastic Differential Equations

> [!info] @songScoreBasedGenerativeModeling2021 · Song et al. · 2021
> [Open in Zotero](zotero://select/library/items/FUTSN73A) · [DOI](https://doi.org/10.48550/arXiv.2011.13456) · [URL](http://arxiv.org/abs/2011.13456) · [PDF](file:///home/lonper/Zotero/storage/BA6XFI9H/Song%20等%20-%202021%20-%20Score-Based%20Generative%20Modeling%20through%20Stochastic%20Differential%20Equations.pdf)

## Abstract

> [!abstract]- Click to expand
> Creating noise from data is easy; creating data from noise is generative modeling. We present a stochastic differential equation (SDE) that smoothly transforms a complex data distribution to a known prior distribution by slowly injecting noise, and a corresponding reverse-time SDE that transforms the prior distribution back into the data distribution by slowly removing the noise. Crucially, the reverse-time SDE depends only on the time-dependent gradient field (\aka, score) of the perturbed data distribution. By leveraging advances in score-based generative modeling, we can accurately estimate these scores with neural networks, and use numerical SDE solvers to generate samples. We show that this framework encapsulates previous approaches in score-based generative modeling and diffusion probabilistic modeling, allowing for new sampling procedures and new modeling capabilities. In particular, we introduce a predictor-corrector framework to correct errors in the evolution of the discretized reverse-time SDE. We also derive an equivalent neural ODE that samples from the same distribution as the SDE, but additionally enables exact likelihood computation, and improved sampling efficiency. In addition, we provide a new way to solve inverse problems with score-based models, as demonstrated with experiments on class-conditional generation, image inpainting, and colorization. Combined with multiple architectural improvements, we achieve record-breaking performance for unconditional image generation on CIFAR-10 with an Inception score of 9.89 and FID of 2.20, a competitive likelihood of 2.99 bits/dim, and demonstrate high fidelity generation of 1024 x 1024 images for the first time from a score-based generative model.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
- omg你难道不觉得数学化这个玩意很神圣吗，比起简单的拟人
- omg你难道不觉得统一化这个玩意很神圣吗，比起简单的灵光一闪
- omg你难道不觉得看高手写论文很享受吗
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
### Imported 2026-05-17 15:24

- 🟡 **p.1** — Crucially, the reverse-time SDE depends only on the time-dependent gradient field (a.k.a., score) of the perturbed data distribution. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=1&annotation=K7RVNG9R)
  - 💬 *我的批注*：$dx = \left[ f(x,t) - g(t)^2 \nabla_x \log p_t(x) \right] dt + g(t)\, d\bar{w}$

### Imported 2026-05-18 16:20

- 🟡 **p.1** — By leveraging advances in score-based generative modeling, we can accurately estimate these scores with neural networks, and use numerical SDE solvers to generate samples. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=1&annotation=78T8DFNK)
  - 💬 *我的批注*：$score \approx \frac{1}{\sigma_t}\epsilon_\theta(x_t,t).$ 

- 🟡 **p.1** — In particular, we introduce a predictor-corrector framework to correct errors in the evolution of the discretized reverse-time SDE. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=1&annotation=AXEJQACM)

- 🟡 **p.1** — We also derive an equivalent neural ODE that samples from the same distribution as the SDE, but additionally enables exact likelihood computation, and improved sampling efficiency. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=1&annotation=PIDUSE4P)
  - 💬 *我的批注*：$dx = \left[ f(x,t) - \frac{1}{2} g(t)^2 \nabla_x \log p_t(x)\right] dt$ DDIM like

- 🟡 **p.1** — For continuous state spaces, the DDPM training objective implicitly computes scores at each noise scale. We therefore refer to these two model classes together as score-based generative models. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=1&annotation=J5YEZB47)

- 🟡 **p.2** — Solving a reversetime SDE yields a score-based generative model. Transforming data to a simple noise distribution can be accomplished with a continuous-time SDE. This SDE can be reversed if we know the score of the distribution at each intermediate time step, ∇x log ptpxq. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=2&annotation=8LGCKZD5)

- 🟡 **p.2** — (i) Predictor-Corrector (PC) samplers that combine numerical SDE solvers with score-based MCMC approaches, such as Langevin MCMC (Parisi, 1981) and HMC (Neal et al., 2011); and (ii) deterministic samplers based on the probability flow ordinary differential equation (ODE). [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=2&annotation=A9L5I47F)

- 🟡 **p.2** — We can modulate the generation process by conditioning on information not available during training, because the conditional reverse-time SDE can be efficiently estimated from unconditional scores. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=2&annotation=3BYSML5B)

- 🟡 **p.2** — The methods of SMLD and DDPM can be amalgamated into our framework as discretizations of two separate SDEs. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=2&annotation=JAMMFZAF)

- 🟡 **p.3** — θ ̊ “ arg min  θ  N  ÿ  i“1  σ2  i EpdatapxqEpσi px ̃|xq  “ ‖sθp ̃x, σiq  ́ ∇x ̃ log pσi px ̃ | xq‖2  2  ‰. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=3&annotation=AY4H7G47)
  - 💬 *我的批注*：虽然监督信号是条件 score：  
        
      $$  
      \nabla_{\tilde{x}} \log p_{\sigma_i}(\tilde{x}\mid x),  
      $$  
        
      但最优网络学到的是边缘 score：  
        
      $$  
      \nabla_x \log p_{\sigma_i}(\tilde{x}).  
      $$  
        
      原因是 denoising score matching 有恒等式：  
        
      $$  
      \nabla_{\tilde{x}} \log p_\sigma(\tilde{x})  
      =  
      \mathbb{E}_{p(x\mid \tilde{x})}  
      \left[  
      \nabla_{\tilde{x}} \log p_\sigma(\tilde{x}\mid x)  
      \right].  
      $$  
      (bayes推)

- 🟡 **p.3** — the optimal score-based model sθ ̊ px, σq matches  ∇x log pσpxq almost everywhere for σ P tσiuN  i“1 [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=3&annotation=MUG7XY64)

### Imported 2026-05-18 17:11

- 🟢 **p.2** — DENOISING SCORE MATCHING WITH LANGEVIN DYNAMICS (SMLD) [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=2&annotation=DB4QAGE6)
  - 💬 *我的批注*：主要有两个部分：  
      1. training  
      ```  
      Input:  
          Dataset D  
          Noise scales σ1 < σ2 < ... < σN  
          Score network sθ(x, σ)  
          Optimizer  
          Number of training steps K  
        
      For step = 1 to K:  
        
          1. Sample clean data:  
                 x ~ D  
          2. Sample a noise scale:  
                 i ~ Uniform({1, 2, ..., N})  
                 σ = σi  
          3. Add Gaussian noise:  
                 z ~ N(0, I)  
                 x_tilde = x + σ z  
          4. Compute denoising score matching target:  
                 target = ∇_{x_tilde} log pσ(x_tilde | x)  
                        = -(x_tilde - x) / σ^2  
                        = -z / σ  
          5. Compute weighted DSM loss:  
                 loss = σ^2 || sθ(x_tilde, σ) - target ||_2^2  
             equivalently:  
                 loss = σ^2 || sθ(x + σz, σ) + z / σ ||_2^2  
          6. Update θ by gradient descent:  
                 θ ← θ - η ∇θ loss  
        
      Output:  
          Trained score network sθ(x, σ)  
      ```  
        
      2. sampling：  
      ```  
      Input:  
          Trained score network sθ(x, σ)  
          Noise scales σ1 < σ2 < ... < σN  
          Step sizes ε1, ε2, ..., εN  
          Number of Langevin steps per noise scale M  
        
      1. Initialize from the largest noise distribution:  
             x_N^0 ~ N(0, σN^2 I)  
        
      2. For i = N, N-1, ..., 1:  
             If i < N:  
                 x_i^0 = x_{i+1}^M  
        
             For m = 1, 2, ..., M:  
                 Sample Gaussian noise:  
                     z_i^m ~ N(0, I)  
        
                 Langevin update:  
                     x_i^m  
                     =  
                     x_i^{m-1}  
                     + ε_i sθ(x_i^{m-1}, σ_i)  
                     + sqrt(2 ε_i) z_i^m  
        
      3. Return:  
             x_1^M  
      ```

### Imported 2026-05-20 11:30

- 🟡 **p.3** — θ ̊ “ arg min  θ  N  ÿ  i“1  p1  ́ αiqEpdatapxqEpαi px ̃|xqr‖sθpx ̃, iq  ́ ∇x ̃ log pαi px ̃ | xq‖2  2s. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=3&annotation=LMNWYJC7)
  - 💬 *我的批注*：$$\nabla_{\tilde{x}} \log p_{\alpha_i}(\tilde{x}\mid x) = -\frac{\epsilon}{\sqrt{1-\alpha_i}}.$$

- 🟡 **p.3** — Our goal is to construct a diffusion process txptqutT“0 indexed by a continuous time variable t P r0, T s,  such that xp0q „ p0, for which we have a dataset of i.i.d. samples, and xpT q „ pT , for which we have a tractable form to generate samples efficiently. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=3&annotation=XF2LHSPF)

- 🟡 **p.4** — Once the score of each marginal distribution, ∇x log ptpxq, is known for all t, we can derive the reverse diffusion process from Eq. (6) and simulate it to sample from p0. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=4&annotation=WL95YAR8)

- 🟡 **p.4** — θ ̊ “ arg min  θ  Et  !  λptqExp0qExptq|xp0q  “∥  ∥sθpxptq, tq  ́ ∇xptq log p0tpxptq | xp0qq  ∥ ∥  2 2  ‰ )  . (7) [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=4&annotation=ASJFKQJA)

- 🟡 **p.4** — λ91{E“ ∥  ∥∇xptq log p0tpxptq | xp0qq  ∥ ∥  2 2  ‰ [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=4&annotation=7IWZT48L)

- 🟡 **p.5** — We typically need to know the transition kernel p0tpxptq | xp0qq to efficiently solve Eq. (7). [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=5&annotation=KMEPBCCM)
  - 💬 *我的批注*：神秘的积分小方法

- ⚫ **p.5** — For more general SDEs, we may solve Kolmogorov’s forward equation (Øksendal, 2003) to obtain p0tpxptq | xp0qq. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=5&annotation=W6B88UX8)
  - 💬 *我的批注*：hard to solve

- 🟡 **p.5** — Due to this difference, we hereafter refer to Eq. (9) as the Variance Exploding (VE) SDE, and Eq. (11) the Variance Preserving (VP) SDE. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=5&annotation=BTM6ALIU)

- 🟡 **p.5** — Inspired by the VP SDE, we propose a new type of SDEs which perform particularly well on likelihoods (see Section 4.3), given by  dx “  ́ 1  2 βptqx dt `  b  βptqp1  ́ e ́2 şt  0 βpsqdsqdw. (12) [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=5&annotation=YMX7LURH)

- 🟡 **p.6** — have a score-based model sθ ̊ px, tq « ∇x log ptpxq, [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=6&annotation=GMJJR4WN)

- 🟡 **p.6** — sample from pt directly, and correct the solution of a numerical SDE solver. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=6&annotation=VASVAEXU)

- 🟡 **p.7** — dx “  ”  f px, tq  ́ 1  2 gptq2∇x log ptpxq  ı  dt, (13) [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=7&annotation=CBUGP4E3)
  - 💬 *我的批注*：保证 \frac{\partial p_t(x)}{\partial t}=-\nabla_x \cdot \left( f(x,t)p_t(x) \right)+\frac{1}{2} g(t)^2 \Delta_x p_t(x).不变

- 🟡 **p.8** — 4.4 ARCHITECTURE IMPROVEMENTS [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=8&annotation=GNKS6T2X)
  - 💬 *我的批注*：改了u-net之类的

- 🟡 **p.9** — dx “ tf px, tq  ́ gptq2r∇x log ptpxq ` ∇x log ptpy | xqsudt ` gptqdw ̄ . [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=9&annotation=C4QFBTG9)
  - 💬 *我的批注*：由$$dx=\left[f(x,t)-g(t)^2 \nabla_x \log p_t(x\mid y)\right] dt+g(t)\, d\bar{w}.$$推导来

- 🟡 **p.9** — In some cases, it is possible to train a separate model to learn the forward process log ptpy | xptqq and compute its gradient. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=9&annotation=XB2LP7LY)

- 🟡 **p.9** — When y represents class labels, we can train a time-dependent classifier ptpy | xptqq for class-conditional sampling. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=9&annotation=YM95G9BY)

- 🟡 **p.9** — Afterwards, we may employ a mixture of cross-entropy losses over different time steps, like Eq. (7), to train the time-dependent classifier ptpy | xptqq. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=9&annotation=YI2A66WV)

- 🟡 **p.9** — Imputation is a special case of conditional sampling. [⤴](zotero://open-pdf/library/items/BA6XFI9H?page=9&annotation=2K3UKY39)
  - 💬 *我的批注*：分开unknown与known

%% end annotations %%

## 我的总结（读完后填）

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
1. 物理图像构建：来个最重要的公式链，从上可以推到下$$dx = f(x,t)\,dt + g(t)\,dw.$$$$ \frac{\partial p_t(x)}{\partial t}=-\nabla_x \cdot \left( f(x,t)p_t(x) \right)+\frac{1}{2} g(t)^2 \Delta_x p_t(x).$$$$dx =\left[f(x,t) - g(t)^2 \nabla_x \log p_t(x)
\right] dt+ g(t)\, d\bar{w}$$大致来说，第1,3个对应sampling，第2个对应training.第3个$dt<0$对应反向过程。
2. 来个概率图解释一下公式，本来应该是$C\times H\times W$维度的，简化为一维。以DDPM为例，$f=-\frac{1}{2}\beta(t) x$，$g= \sqrt{\beta(t)}$。每个x不断的向0位移，并且随机游走。反应到整体上就是如下图前向扩散所示。反向过程则是一边向远离0移动，一边朝该时段的高概率方向移动，一边少量随机游走。![[songScoreBasedGenerativeModeling2021-1779104321502.webp]]
3. 然后score怎么拟合，可以使用这个等式 $$\nabla_{\tilde{x}} \log p_\sigma(\tilde{x})       =     \mathbb{E}_{p(x\mid \tilde{x})}  
      \left[        \nabla_{\tilde{x}} \log p_\sigma(\tilde{x}\mid x)  \right]. $$然后$$\begin{aligned}&E_{x,\tilde{x}}\left(\epsilon(\tilde{x})-\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\right)^2\\={}&E_{\tilde{x}}E_{x\mid\tilde{x}}\left(\epsilon(\tilde{x})-\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\right)^2\\={}&E_{\tilde{x}}E_{x\mid\tilde{x}}\Big(\epsilon(\tilde{x})^2-2\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\,\epsilon(\tilde{x})+\big(\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\big)^2\Big)\\={}&E_{\tilde{x}}\Big(\epsilon(\tilde{x})^2-2E_{x\mid\tilde{x}}\big(\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\big)\,\epsilon(\tilde{x})\\&\qquad\qquad+E_{x\mid\tilde{x}}\Big(\big(\nabla_{\tilde{x}}\log p_\sigma(\tilde{x}\mid x)\big)^2\Big)\Big)\end{aligned}$$作为loss即可拟合
4. 具体流程：
	1. Step 1：选择 forward SDE
	2. Step 2：写出转移核 $p_{0t}\bigl(x(t)\mid x(0)\bigr)$
	3. Step 3：计算条件 score 作为训练目标
	4. Step 4：训练 score network$$\theta^*=\arg\min_{\theta}\mathbb{E}_t\left[\lambda(t)\mathbb{E}_{(0)}\mathbb{E}_{x(t)\mid x(0)}\left[\left\|s_\theta(x(t),t)-\nabla_{x(t)}\log p_{0t}(x(t)\mid x(0))\right\|_2^2\right]\right].$$
	5. Step 5：采样时构造 reverse-time SDE
5. sampling方法：
	* 除了直接reverse,注意到我们还有一个$score=\nabla_x \log p_t(x)$可以用来改善sampling
	* 也就是说，我们本来希望$x_0$坍缩到尖峰附近去，但是万一reverse时$dw$给大伙一个小惊喜，变成硕大的噪声，或者说离散化导致一些偏差，我们可以通过score来让采样点更加靠近尖峰。
	* 流程为：
		1. 使用$$dx =\left[f(x,t) - g(t)^2 \nabla_x \log p_t(x)\right] dt+ g(t)\, d\bar{w}$$的离散化来predict
		2. 使用如$$dx=\nabla_x \log p_t(x)\, d\tau+\sqrt{2}\, dw_\tau .$$的离散化来correct，通常做m步。该Langevin MCMC过程的$\frac{\partial p_t(x)}{\partial t}$为0,相当于不改变时间分布，不会影响predictor的时间。
	* 它通常比单纯把 predictor 步数翻倍、但不加入 corrector 的 P2000 更好。
	* 还有一种确定性predict方法，同样是保证$$ \frac{\partial p_t(x)}{\partial t}=-\nabla_x \cdot \left( f(x,t)p_t(x) \right)+\frac{1}{2} g(t)^2 \Delta_x p_t(x).$$不变的情况下，设反向过程为$$dx=\left[f(x,t)-\frac{1}{2} g(t)^2 \nabla_x \log p_t(x)\right] dt,$$
6. 条件sampling：核心公式是条件反向 SDE：$$	dx	=	\left\{	f(x,t)	-	g(t)^2	\left[	\nabla_x \log p_t(x)	+	\nabla_x \log p_t(y\mid x)	\right]	\right\}dt	+	g(t)\, d\bar{w}.	$$	
	它来自贝叶斯公式：	$$	\nabla_x \log p_t(x\mid y)	=	\nabla_x \log p_t(x)	+	\nabla_x \log p_t(y\mid x).	$$
	其中：
	
	- $\nabla_x \log p_t(x)$：无条件 score，由 score network 提供；
	- $\nabla_x \log p_t(y\mid x)$：条件引导项，由分类器、观测模型、启发式或领域知识提供。
7. 总之，记住两个图像就行：
	1. 概率变化图像
	2. 采样点变化图像
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
-
-
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
- [ ]
- [ ]
- [ ]
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/songScoreBasedGenerativeModeling2021.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/songScoreBasedGenerativeModeling2021.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-05-20T11:30:34.333+08:00 %%
