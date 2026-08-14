---
type: concept
title: Flow Matching（FM）
aliases: [Flow Matching, FM, 流匹配]
tags: [flow-matching, cnf, generative-model, ode]
status: active
created: 2026-05-24
updated: 2026-08-14
sources: ["[[wiki/sources/lipmanFlowMatchingGenerative2023]]", "[[wiki/sources/liuFlowStraightFast2022a]]", "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]", "[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]", "[[wiki/sources/chaTrainingFreeRefinementFlow2026]]", "[[wiki/sources/shaulBespokeSolversGenerative2023]]", "[[wiki/sources/shaulBespokeNonStationarySolvers2024]]", "[[wiki/sources/wangTamingRectifiedFlow2025]]", "[[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026]]", "[[wiki/sources/chenBiAnchorInterpolationSolver2026]]", "[[wiki/sources/flow-matching-solver-method-taxonomy]]", "[[wiki/sources/galashovLearnGuideYour2025]]"]
evidence: ["[[research/experiments/2026-08-02-reject-and-skip-toy-report]]", "[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]"]
---

# Flow Matching（FM）

> 概念主页。源：[[wiki/sources/lipmanFlowMatchingGenerative2023|Lipman et al. 2023, ICLR]]。
> ⚠️ 时间约定：FM 中 $t=0$ 噪声、$t=1$ 数据，生成沿 $t:0\to1$ 正向；与本 vault 的 [[wiki/methods/ddpm|DDPM]]/[[wiki/concepts/score-sde|Score SDE]]（$t:0\to T$ 数据→噪声）**相反**。

## 一句话定义

不构造加噪 SDE，而是**直接选一条 probability path $p_t$（噪声↔数据）、用网络回归它的生成向量场 $v_t$**，再沿 ODE $\dot x=v_t(x)$ 生成。这给出 simulation-free（训练时不解 ODE）地训练 [[wiki/concepts/continuous-normalizing-flow|CNF]] 的办法。

## 与 score-based 的根本分工

| | score-based（[[wiki/concepts/score-sde]]） | Flow Matching |
|---|---|---|
| 先验对象 | 一条加噪 **SDE**（$v=f$ 解析已知） | 一条 **probability path** $p_t$ |
| 网络学什么 | score $\nabla_x\log p_t$ | 速度场 $v_t(x)$ |
| 采样 | 反向 SDE / [[wiki/concepts/probability-flow-ode\|PF-ODE]] | ODE $\dot x=v_t$ |
| 关于 $p(x)$ 的知识 | 在 score 里 | **全在 $v(x,t)$ 里** |

> 一句话延续 vault 既有提法：**DDIM/PF-ODE = diffusion 的训练 + flow 的采样；FM = 连训练也 flow 化**。

> 📐 **更深一层（对象本质）**：score $\nabla\log p_t$ 必是**保守场**（梯度），$v_t$ 一般是**非保守场**（写不成 $\nabla(\cdot)$）——两者只在固定高斯路径下可线性互转。这是 FM 更一般、OT 路径能"走直线"的数学根源，详见 [[wiki/comparisons/score-vs-velocity-field]]。

## 朴素目标与它的麻烦

$$\mathcal L_{\mathrm{FM}}(\theta)=\mathbb E_{t,\,p_t(x)}\big\|v_t(x;\theta)-u_t(x)\big\|^2.$$
但边缘 $p_t,u_t$ 无闭式、不可采样 → intractable。解法是 [[wiki/concepts/conditional-flow-matching|Conditional Flow Matching]]：用 per-example 条件场作回归目标，**梯度与上式相同**（同 [[wiki/concepts/score-matching|DSM]] 套路）。

## 自洽性判据：连续性方程

$(p_t,v_t)$ 相容 ⟺ 满足
$$\frac{\partial p_t}{\partial t}+\nabla\!\cdot(p_t v_t)=0,$$
即 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]] 在无扩散项（$g=0$）时的退化——FM 是确定性流，没有 $\tfrac12 g^2\Delta p$。

## 为什么重要

1. **路径成了可设计对象**：diffusion 路径只是 Gaussian 路径族的特例；可换上 [[wiki/concepts/optimal-transport-path|OT 直线路径]]，训练/采样更快。
2. **真正换掉了训练目标**却不破坏"迭代生成 + 预测速度场 + 沿链注入条件"的范式——是 [[wiki/overview]] 可变性光谱里"训练目标可演化、范式不变"的关键样本。
3. **采样省**：用现成自适应 ODE solver，NFE 低、训练期采样成本恒定。

## 与其他概念的关系

- 训练技巧：[[wiki/concepts/conditional-flow-matching]]（核心）
- 载体：[[wiki/concepts/continuous-normalizing-flow]]
- 路径实例：[[wiki/concepts/optimal-transport-path]]、diffusion 路径（[[wiki/methods/ddpm]]/[[wiki/methods/ncsn]]）
- 采样近亲：[[wiki/concepts/probability-flow-ode]]（同为确定性 ODE，但 score 事后导出 vs FM 直接训练）
- 自洽判据：[[wiki/concepts/fokker-planck-equation]]（无扩散退化）
- learned guidance：[[wiki/methods/learn-to-guide|Learn to Guide]] 冻结 1.05B velocity model 学习 $\omega(c,s,t)$；它不改变 FM 训练目标或 ODE solver。

## 在 text-guided editing 中的作用

- FM 是 SD3 / FLUX 等 [[wiki/methods/rectified-flow|rectified-flow]] 一线的训练底座；这些模型上的编辑方法（如 RF-Inversion、✅ [[wiki/methods/flux-kontext|FLUX.1 Kontext]]）或在 ODE 链上注入条件、或走 [[wiki/concepts/in-context-conditioning|in-context token 拼接]]。
- ⚠️ FM 的 $t$ 与 diffusion 的 $t$ 方向相反，做"介入时间步"分析时须先统一坐标。

## 出处与引用

- [[wiki/sources/lipmanFlowMatchingGenerative2023]]（FM 原文）
- [[wiki/sources/galashovLearnGuideYour2025]]（冻结 1.05B FM backbone 的 learned guidance）
- 并行工作：[[wiki/sources/liuFlowStraightFast2022a|Liu et al. 2022 (Rectified Flow)]]（已 ingest 2026-05-26；公式上 RF = FM-OT 路径取 $\sigma_{\min}=0$ + 任意 coupling 接口 + [[wiki/concepts/reflow|reflow]]）、[[wiki/concepts/stochastic-interpolants|Stochastic Interpolants]]（[[wiki/sources/albergoStochasticInterpolants2023|Albergo, Boffi & Vanden-Eijnden 2023]]，✅ 已 ingest——"自由选 interpolant + 选 diffusion 系数"统一 flow 与 diffusion，FM 是其 ODE 侧特例）
- **bridge ODE vs bridge SDE**：FM/RF 是**确定 ODE**地连接两端分布（学速度场）；[[wiki/methods/ddbm|DDBM]] 是**随机 SDE** 版的 [[wiki/concepts/diffusion-bridge|扩散桥]]（学 score、[[wiki/concepts/doob-h-transform|Doob h]] 钉端点）。DDBM 论文称"unifies OT-FM"，但其正文表明这只是 noiseless 极限 $c\to0$ + 特定 VE schedule 的**有条件约化**（非严格特例）；两者真正的统一归宿是 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]，且 DDBM 用的是不同的 denoising bridge score-matching loss
- **速度平均化与推理时修正**：
  - [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF（Zhang et al. 2025）]]：在 velocity space 再跑一层 RF 学加速度，training-based 路线，代价大且高维收益低
  - [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS（Cha et al. 2026）]]：用 $\nabla_x \cdot u_\theta$ 做 discrepancy proxy，inference-time spatial refinement，training-free plug-and-play，直接超过 HRF
  - [[wiki/concepts/reject-and-skip]]：研究中的 inference-time temporal coupling intervention；检测局部不可信 trial 后回滚并搜索远端可信出口。二维 oracle toy 支持机制存在性，但官方 1-RF 上尚未完成非 oracle 机制验证与等 NFE 分布评测
- **ODE solver 改进谱系**（详见 [[wiki/concepts/ode-solver-taxonomy]]）：FM 第 3 条优点"用现成 ODE solver"后续被大量工作精炼——
  - [[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solver（2023）]]：scale-time 变换参数化，~80 params 离线优化
  - [[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS（2024）]]：Non-Stationary solver 族，严格超集前作，<200 params；证明经典 RK/DDIM/指数积分器都是其特例
  - [[wiki/sources/wangTamingRectifiedFlow2025|RF-Solver（2025）]]：training-free 二阶 Taylor 展开，误差 $O(h^2) \to O(h^3)$，面向 inversion 和 editing
  - [[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026|FastFlow（2026）]]：有限差分 velocity 外推 + UCB bandit 选跳步长度，training-free 自适应省 NFE
  - [[wiki/sources/chenBiAnchorInterpolationSolver2026|BA-solver（Chen et al. 2026）]]：冻结 backbone + 轻量 SideNet（~6M params）做区间内速度插值 + 双 anchor + Gauss–Lobatto 求积，7 NFE FID 1.96（ImageNet-256）；属"learnt velocity interpolator"新档
  - 这些 solver 改进都不处理速度平均化问题——它们假设 $v_\theta$ 本身正确，只优化离散化方案或计算分配
  - **成本口径 caveat**：[[wiki/sources/flow-matching-solver-method-taxonomy|用户整理的 solver 分类]]进一步区分步数、backbone NFE、全局阶数、自适应拒步、历史缓存和隐藏计算。积分公式、时间网格、辅助网络、轨迹选择与系统并行属于不同层，不能只按名义 NFE 横比
