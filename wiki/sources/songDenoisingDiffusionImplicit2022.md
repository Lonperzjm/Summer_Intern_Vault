---
type: source
title: "Denoising Diffusion Implicit Models (DDIM)"
aliases: [DDIM, "Song, Meng & Ermon 2021", songDenoisingDiffusionImplicit2022]
tags: [diffusion, sampling-acceleration, deterministic-sampling, foundational]
status: stable
created: 2026-05-14
updated: 2026-05-26
raw: "[[raw/literature-notes/songDenoisingDiffusionImplicit2022]]"
authors: [Jiaming Song, Chenlin Meng, Stefano Ermon]
venue: ICLR
year: 2021
arxiv: "2010.02502"
---

# Denoising Diffusion Implicit Models (DDIM)

> Song, Meng, Ermon · [[wiki/entities/stanford|Stanford]] · ICLR 2021（arXiv 2020.10）· [arXiv 2010.02502](https://arxiv.org/abs/2010.02502)
> 原始文献笔记：[[raw/literature-notes/songDenoisingDiffusionImplicit2022]]
> 注：Zotero 把 itemType 记为 `preprint` / `year: 2022`（取自最后修订年）；本页统一采用正式发表信息 **ICLR 2021**。

## Motivation

[[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]] 样本质量好，但采样必须串行模拟一条长度 $T$（默认 1000）的马尔可夫链——每一步都要过一次网络，且步与步之间不能并行。这让 DDPM 在算力受限、延迟敏感的场景下不实用，采样比 GAN / 自回归慢一个数量级。

作者要回答：**能不能在不重新训练的前提下，把采样步数砍掉一两个数量级？** 关键观察是——DDPM 的训练目标 $L_\text{simple}$ 实际上**只依赖前向的边缘分布** $q(x_t\mid x_0)$，并不依赖"前向必须是马尔可夫链"这个假设。

## Method

### 非马尔可夫前向过程族（核心构造）

DDIM 定义一族由方差参数 $\sigma\in\mathbb R^T_{\ge 0}$ 索引的**非马尔可夫**推断过程 $q_\sigma$，其设计约束是：对所有 $t$，边缘分布与 DDPM 完全相同

$$
q_\sigma(x_t\mid x_0) = \mathcal N(\sqrt{\bar\alpha_t}\,x_0,\ (1-\bar\alpha_t)I)
$$

为满足这一约束，前向被写成直接以 $x_0$ 为条件的形式（不再是逐步的 $q(x_t\mid x_{t-1})$）。对应的反向转移有闭式：

$$
q_\sigma(x_{t-1}\mid x_t, x_0) = \mathcal N\!\left(\sqrt{\bar\alpha_{t-1}}\,x_0 + \sqrt{1-\bar\alpha_{t-1}-\sigma_t^2}\cdot\frac{x_t-\sqrt{\bar\alpha_t}\,x_0}{\sqrt{1-\bar\alpha_t}},\ \sigma_t^2 I\right)
$$

> 概念页：[[wiki/concepts/non-markovian-diffusion]]。

### 训练目标不变（Theorem 1）

> **Theorem 1.** 对所有 $\sigma>0$，存在 $\gamma\in\mathbb R^T_{>0}$ 与 $C\in\mathbb R$，使得 $J_\sigma = L_\gamma + C$。

也就是说，整个非马尔可夫族共享同一个变分目标（在 $t$ 之间不共享参数的前提下，与 DDPM 的 $L_\text{simple}$ 等价）。**含义：一个训好的 [[wiki/concepts/epsilon-parameterization|ε 网络]]，无需任何重训，就同时是这一整族生成过程的解。** $\sigma$ 是一个纯采样期旋钮。

### 生成过程：从 $x_t$ 走到 $x_{t-1}$

先用网络从 $x_t$ 预测出 $x_0$（记 $f_\theta^{(t)}(x_t)=\frac{x_t-\sqrt{1-\bar\alpha_t}\,\varepsilon_\theta(x_t,t)}{\sqrt{\bar\alpha_t}}$），再代回上式采样 $x_{t-1}$：

$$
x_{t-1} = \sqrt{\bar\alpha_{t-1}}\underbrace{\left(\frac{x_t-\sqrt{1-\bar\alpha_t}\,\varepsilon_\theta(x_t,t)}{\sqrt{\bar\alpha_t}}\right)}_{\text{预测的 }x_0}
+ \underbrace{\sqrt{1-\bar\alpha_{t-1}-\sigma_t^2}\cdot\varepsilon_\theta(x_t,t)}_{\text{指向 }x_t\text{ 的方向}}
+ \sigma_t z_t
$$

- $\sigma_t = \sqrt{\tilde\beta_t}$（即 $\sqrt{(1-\bar\alpha_{t-1})/(1-\bar\alpha_t)}\sqrt{1-\bar\alpha_t/\bar\alpha_{t-1}}$）→ 生成过程退回 **DDPM**。
- $\sigma_t = 0$ → 生成过程**完全确定性**，前向不再随机——这就是 **DDIM**（"implicit"：从隐变量到样本是一个隐式确定映射）。

### 加速采样（4.2）

既然前向非马尔可夫，反向过程可以只定义在 $[1,\dots,T]$ 的**任意子序列** $\tau=(\tau_1,\dots,\tau_S)$ 上，用同样的更新式在 $x_{\tau_i}\to x_{\tau_{i-1}}$ 之间跳步。同一个网络、同一套 $\bar\alpha$，只是少跑几步。这对 DDPM、DDIM 以及该族中所有过程都适用。

### ODE 视角

把 DDIM 更新改写、令步长趋于 0，得到一个常微分方程

$$
\mathrm d\bar x(t) = \varepsilon_\theta^{(t)}\!\left(\frac{\bar x(t)}{\sqrt{\sigma^2+1}}\right)\mathrm d\sigma(t),\qquad \sigma = \sqrt{1-\bar\alpha}/\sqrt{\bar\alpha}
$$

DDIM 即该 ODE 的一种离散化。两个推论：(i) 步数越多离散误差越小、质量越高；(ii) 该 ODE **可反向积分**——把 $x_0$ 编码回 $x_T$，即 **DDIM inversion** 的雏形。与 [[wiki/concepts/score-sde|Score SDE]] 的 probability-flow ODE 是同一脉络。

## Results

- **加速**：DDIM 在 **20–100 步**内即可产出与 1000 步 DDPM 相当质量的样本，按 wall-clock 计 **10×–50×** 加速；步数与质量之间是平滑可调的 trade-off（增大 $\dim(\tau)$ 质量单调上升）。
- **consistency**：$\sigma=0$ 时 $x_T$ 唯一决定 $x_0$；固定 $x_T$、改变采样步数，得到的样本共享高层特征——DDPM（$\sigma>0$）不具备此性质。
- **latent space 语义插值**：在 $x_T$ 空间做球面插值，能得到语义上平滑过渡的图像，说明 $x_T$ 是一个有意义的隐空间。
- **重建**：DDIM 可把样本编码回 latent 再解码回来，重建误差小——为下游 inversion-based 编辑铺路。

## 关系

- 实体：[[wiki/entities/jiaming-song]]（第一作者）、[[wiki/entities/stefano-ermon]]（资深作者）、[[wiki/entities/stanford]]（机构）；Chenlin Meng（二作）暂无 entity 页
- 方法：[[wiki/methods/ddim]]（本论文 = 该方法的奠基）
- 概念：
  - [[wiki/concepts/non-markovian-diffusion]] —— 本文核心构造
  - [[wiki/concepts/epsilon-parameterization]] —— DDIM 复用 DDPM 同一 ε 网络的根本原因
  - [[wiki/concepts/diffusion-process]] —— DDIM 是对"前向必须马尔可夫"假设的松绑
  - [[wiki/concepts/score-sde]] —— DDIM 的 ODE 视角 = probability-flow ODE 的离散化
  - [[wiki/concepts/variational-bound-elbo]] —— Theorem 1 中 $J_\sigma$ 即此变分目标的非马尔可夫版本
- 上游：[[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]]（共享训练目标与 ε 网络）
- 基准：[[wiki/benchmarks/cifar10]]、[[wiki/benchmarks/lsun]]（论文在二者上报告分步数 FID，展示 step–quality trade-off）
- 下游：DDIM inversion → 几乎所有 inversion-based text-guided editing 方法；Score SDE 的 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]；[[wiki/methods/rectified-flow|Rectified Flow]]（[[wiki/sources/liuFlowStraightFast2022a|Liu et al. 2022]]）把同一条确定性 ODE 家谱从"事后导出"推到"训练阶段主动拉直"（[[wiki/concepts/reflow|reflow]]）；扩散蒸馏 / Consistency Models 等加速线

## 对我的 thesis 的启示

- **直接验证 [[wiki/overview]] 推论 3"采样速度是开放赛道"**：DDIM 证明了"加速可以是纯采样期的事，无需碰训练"——这把"采样加速"从训练问题降级为推断问题，对编辑场景（本就要反复采样）尤其关键。
- **强化可变性光谱**：DDIM 不动训练目标、不动 backbone，只动"采样链的形状（步数、$\sigma$）"——属于光谱里"研究杠杆"那一档（介入方式 / 采样几何），且代价极低。这是"范式不变、组件可调"的又一个干净样本。
- **ODE 反向 = inversion 的理论入口**：DDIM 把"编辑 = 在确定性轨迹上注入条件"变得可操作（先 invert 到 $x_T$，再带条件 denoise）。overview「主要派系」里的 inversion-based 一类，其技术底座就在这里。
- **关于我笔记里第 2 条 idea（改进加噪 / 分段选噪声）**：这个想法实际上正落在 DDIM"非马尔可夫前向族"的延长线上——DDIM 已经说明"前向过程不必马尔可夫、只要边缘对齐就共享目标"。可探索的增量是：在子序列 $\tau$ 与 $\sigma$ 之外，是否还有别的自由度（如非均匀 / 数据自适应的 $\tau$ 选择）能再压步数。记入 open questions。

## Open questions / 待追

- [ ] Appendix A：非马尔可夫视角在非高斯情形下的推广（笔记 ⚪ p.3"到时候看看咋回事"）—— 读 PDF Appendix A。
- [ ] ODE 反向积分（$t=0\to T$ 编码）的离散误差有多大？需要多少步才能保证 inversion 可用——这直接决定下游编辑保真度（笔记 🔴 p.6）。
- [ ] $\sigma$ 在 $0$ 与 $\sigma_\text{DDPM}$ 之间的连续取值（论文 $\eta$ 插值）对编辑任务的随机性 / 多样性有何影响。
- [ ] 我的 idea：子序列 $\tau$ 的选择目前是启发式（均匀 / 二次），是否存在数据自适应或可学习的 $\tau$，在固定预算下进一步逼近 1000 步质量。
- [x] DDIM 与 [[wiki/concepts/score-sde|Score SDE]] probability-flow ODE 的精确对应关系——✅ 已对照（[[wiki/sources/songScoreBasedGenerativeModeling2021|Yang Song et al. 2021]] 已 ingest）：DDIM 的确定性采样（$\sigma=0$）即 [[wiki/concepts/probability-flow-ode|PF-ODE]] 的离散特例，二者同源。
