---
type: source
title: "Diffusion Bridge Implicit Models (DBIM)"
aliases: [DBIM, Diffusion Bridge Implicit Models, "Zheng et al. 2025", 隐式扩散桥]
tags: [diffusion-bridge, sampling-acceleration, non-markovian, inversion, score-based]
status: active
created: 2026-06-01
updated: 2026-06-02
raw: "[[raw/literature-notes/zhengDiffusionBridgeImplicit2025]]"
authors: [Kaiwen Zheng, Guande He, Jianfei Chen, Fan Bao, Jun Zhu]
venue: "ICLR 2025 (arXiv 2405.15885)"
year: 2025
arxiv: "2405.15885"
---

# Diffusion Bridge Implicit Models (DBIM)

> 文献笔记：[[raw/literature-notes/zhengDiffusionBridgeImplicit2025]] · arXiv [2405.15885](http://arxiv.org/abs/2405.15885) · Zheng, He, Chen, Bao & Zhu 2025（ICLR）· [code](https://github.com/thu-ml/DiffusionBridge)

## 一句话

**DBIM 之于 [[wiki/methods/ddbm|DDBM]]，正如 [[wiki/methods/ddim|DDIM]] 之于 [[wiki/methods/ddpm|DDPM]]**：把 DDBM 的桥前向松绑成一族**保边缘的[[wiki/concepts/non-markovian-diffusion|非马尔可夫]]桥**，从而**不重训**复用 DDBM 的 bridge score、把采样从数百 NFE 压到 ~20（最多 **25×** 加速）；并诱导一条新颖的桥 ODE。随机性由 $\rho$ 调（$\rho{=}0$ 确定），$\rho{=}0$ 的确定性采样 + **booting noise** 让 DDBM 第一次能做 faithful **encoding / reconstruction / 语义插值**——即 bridge 上的 inversion 入口。

## Motivation

[[wiki/methods/ddbm|DDBM]] 虽好，但采样要模拟 (S)DE、数百次网络评估，太贵；且"在发展高效桥采样器上缺乏理论洞见"（用户 p.3 高亮）。作者要的是**扩散社区那套 training-free 加速配方（DDIM/PF-ODE/高阶 solver）在桥上的对应物**。

## Method

### 1. 非马尔可夫桥族：保边缘 ⇒ 复用 DDBM score、零重训

只要新过程在离散时间步上与 DDBM **共享 N 个边缘** $\{q(x_{t_n}\mid x_T)\}$，就共享训练目标。作者构造一族由方差参数 $\rho$ 索引的桥转移（公式 11）：
$$
q^{(\rho)}(x_{t_n}\mid x_0,x_{t_{n+1}},x_T)=\mathcal N\!\Big(a_{t_n}x_T+b_{t_n}x_0+\sqrt{c_{t_n}^2-\rho_n^2}\,\tfrac{x_{t_{n+1}}-a_{t_{n+1}}x_T-b_{t_{n+1}}x_0}{c_{t_{n+1}}},\ \rho_n^2 I\Big)
$$
**Proposition 3.1（边缘保持）**：$q^{(\rho)}(x_{t_n}\mid x_T)=q(x_{t_n}\mid x_T)$。

> 🟣 **用户的加强结论**：可证更强的 **endpoint-conditioned** 版本 $q^{(\rho)}(x_{t_n}\mid x_0,x_T)=q(x_{t_n}\mid x_0,x_T)$——这使该采样过程可直接用 DDPM 式的 score。（用户 p.4 批注）

因此 DDBM 已训好的 $s_\theta(x_t,t,x_T)\approx\nabla_{x_t}\log q(x_t\mid x_T)$ **原样可用，不需额外训练**。

### 2. 采样更新 = "预测 $x_0$ + 继承噪声 + 可选新噪声"（公式 15）

预测 $\hat x_0=x_\theta(x_{t_{n+1}},t_{n+1},x_T)$，反推 predicted bridge noise
$$
\hat\varepsilon=\frac{x_{t_{n+1}}-a_{t_{n+1}}x_T-b_{t_{n+1}}\hat x_0}{c_{t_{n+1}}},\qquad
x_{t_n}=a_{t_n}x_T+b_{t_n}\hat x_0+\sqrt{c_{t_n}^2-\rho_n^2}\,\hat\varepsilon+\rho_n\varepsilon.
$$
与 [[wiki/methods/ddim|DDIM]] 的更新式同构，只是普通扩散的 $\alpha_t,\sigma_t$ 换成桥的 $a_t,b_t,c_t$。

### 3. $\rho$ = 随机性旋钮；$\rho{=}0$ 确定 + 新 ODE + booting noise

- $\rho_n=0$ ⇒ 无随机、确定性 implicit bridge，诱导一条**新颖的桥 ODE**（启发高阶 solver，文中给到 3 阶）。
- **初始步奇异**（$c_T=0$）：必须引入 **booting noise**。固定 $x_T$ 下它充当 **latent variable**，使 DBIM 能 faithful encoding / reconstruction / 语义插值——这些是 DDBM hybrid sampler（含随机步）与 $\rho>0$ **做不到**的。

## Results

### Image translation（Table 2，FID↓ / LPIPS↓）

| 方法 | NFE | Handbags-64 FID | DIODE-256 FID |
|---|---|---|---|
| I²SB | ≥40 | 7.43 | 9.34 |
| [[wiki/methods/ddbm\|DDBM]] | 118 | 1.83 | 4.43 |
| [[wiki/methods/ddbm\|DDBM]] | 200 | 0.88 | 3.34 |
| **DBIM** | **20** | **1.74** | 4.99 |
| **DBIM** | **100** | 0.89 | **2.57** |

- **DBIM@20 即超过所有 baseline、并 ≈ DDBM@118**；DBIM@100 在 DIODE 上（2.57）**优于 DDBM@200**（3.34）。

### Image inpainting（ImageNet-256，center 128²）

- **DBIM@20 > DDBM@500**（**25× 加速**）；$\eta{=}0$ 在 NFE=100 收敛到 **FID 3.91**（该任务首次 < 4）。

### 🟣 stochasticity 消融（Table 4/5，对 thesis 直接相关）

- **translation**：确定性 $\eta{=}0$ 一致最优。
- **inpainting（更 diverse 的 ImageNet）**：NFE≤20 时 $\eta{=}0$ 近最优；**NFE≥50 时 $\eta{=}0.8\sim1.0$ 反而最优**（随机性提升多样性、在大预算下拿到最佳 FID）。
- 即"**确定性 → 快速收敛 / 可逆；随机性 → 多样性（需大 NFE 才划算）**"——这是"反向桥该注入多少噪声"的一个**有数据的回答**。

## 关系（与已有 wiki 的关联）

- **基座**：[[wiki/methods/ddbm|DDBM]]（DBIM 复用其 bridge score、加速其采样）；本页是 [[wiki/methods/dbim]] 方法页的源。
- **结构对位**：[[wiki/concepts/non-markovian-diffusion|非马尔可夫族]]——DBIM 是它的 **bridge 版**（DDIM:DDPM :: DBIM:DDBM），保边缘、松绑路径、$\rho$ 控随机、子序列跳步。
- **采样**：诱导桥 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]（$\rho{=}0$）+ 高阶 solver；$\eta{=}0$ 收敛到 PF-ODE 的 ground-truth 解。
- **并行（同组）**：[[wiki/synthesis/bridge-sde-editing-landscape|CDBM]]（Consistency Diffusion Bridge Models, NeurIPS'24）——同为 DDBM 加速，但 CDBM 走 consistency **蒸馏/训练**，DBIM 走 **training-free 隐式**；两文共享作者 [[wiki/entities/kaiwen-zheng|Kaiwen Zheng]]、Guande He。
- **概念**：[[wiki/concepts/diffusion-bridge]]、[[wiki/concepts/score-matching]]。
- **人物 / 机构**：[[wiki/entities/kaiwen-zheng]]、Guande He、Jianfei Chen、Fan Bao、[[wiki/entities/jun-zhu]]；[[wiki/entities/tsinghua-university]]。
- **评测**：Edges→Handbags / DIODE-Outdoor / [[wiki/benchmarks/imagenet|ImageNet]] inpainting。

## 对我的 thesis 的启示

<!-- 用户选择（沿用 DDBM ingest 约定）：本节留空，由用户自己写。以下仅列候选角度。 -->

> 本节按惯例**留给你写**，未改 [[wiki/overview]] / [[research/thesis]]。候选挂钩：

- 候选 ①（最重要）：**DBIM 提供了 bridge 上的 inversion 原语**——$\rho{=}0$ 确定 ODE + booting noise 充当 latent，实现 faithful encoding/reconstruction。这填补了随机桥上"inversion 闭合"的空缺；在此基础上做 **text-guided 编辑**仍是开放方向（DBIM 自身只演示 reconstruction / interpolation）。
- 候选 ②：DBIM 的 $\eta$ 消融给了"stochasticity 何时有用"的实证（确定性利于可逆/收敛，随机性利于多样性/大 NFE）——可直接喂给 thesis 的 fidelity↔diversity 论证。

## Open questions / 待追

- [ ] 🔵 booting noise 的分布选择 / 它作为 latent 的语义结构。
- [ ] 高阶桥 solver（≤3 阶）的精确推导与误差。
- [ ] **text-guided editing on DBIM**：DBIM 只演示了 reconstruction/interpolation，**没做 text-driven 编辑**——这正是开放口。
- [ ] DBIM（training-free 隐式）vs CDBM（蒸馏）在编辑/inversion 质量上的权衡。
