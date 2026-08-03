---
type: research
title: 论文 Thesis 演化
status: draft
created: 2026-05-05
updated: 2026-08-03
---

# Thesis

<!-- 我的论文核心论点。在阅读中演化，由 Claude Code 协助维护，每次改动必须经过我确认。 -->

## 当前版本（v0.2 草稿 · 2026-08-03 · Reject-and-Skip）

> 本版本由用户确认以两份实验报告作为当前研究方向后写入。完整证据链见 [[wiki/synthesis/reject-and-skip-research-direction]]，正式实验记录见 [[research/experiments]]。

**研究问题**：在 Flow Matching / Rectified Flow 的 marginal velocity 因多分支条件速度平均而局部不可信时，能否通过非 oracle detector 识别该区域，执行 `trial → reject/rollback → scan → validate/correct → fallback`，以推理时结构化离散偏置主动改变 transport coupling，并在严格相同 NFE 下改善生成分布与失败尾部？

**核心论点（当前工作假设）**：

1. Flow Matching 的最优边缘速度 $v(x,t)=\mathbb E[u_t\mid X_t=x]$ 在分布层面正确，不保证它对有限 NFE 的单条轨迹始终是可信的 sample-wise transport direction。
2. 传统 ODE solver 以逼近 marginal ODE 为目标；[[wiki/concepts/reject-and-skip]] 则把局部异常视为 coupling intervention 的机会：回滚并搜索异常区之后的可信出口，而不是只缩步解析坏场。
3. 2D oracle toy 已给出机制层存在性证据：skip 可显著偏离 RK4，同时降低粗 Euler 的 endpoint outlier 并保留人工 conditional branch。这支持“分布质量、ODE 精度、coupling 行为必须分开评价”。
4. 官方 CIFAR-10 1-RF 已建立真实模型 backbone、数值诊断与强 baseline，但尚未证明内部困难来自 conditional ambiguity，也尚未证明 skip 改善生成分布。因此当前贡献仍处于**机制迁移验证阶段**。

**预期贡献（若后续实验成立）**：

- 一个低额外 NFE 的非 oracle 状态可信度 detector 与 velocity-return 出口搜索；
- 一个包含 rollback、exit validation 和失败回退的 instance-aware inference solver；
- 一套明确区分 ODE accuracy、endpoint distribution 与 sample-wise coupling 的评测协议；
- 在固定 RF backbone、相同总 NFE 下，相对 midpoint/RK4/adaptive Heun/shrink-only 的稳定分布收益。

**诚实边界**：

- Endpoint boundary layer 不能作为主要机制证据；midpoint 已能以标准方法避免精确 $t=1$ 查询。
- Velocity angle、Euler–Heun defect 和 step-doubling 当前只被证明能预测局部数值误差，未被证明能识别 conditional ambiguity。
- Toy 使用 oracle 分支信息且样本量有限，不能直接外推到高维神经模型。
- Reference RMSE 只记录 coupling change；论文级成功必须由多随机种子、严格等 NFE 的 FID/KID、precision/recall、feature OOD 与重尾失败支持。

## 历史版本 v0.1（2026-06-01 · Bridge-SDE · 后于 2026-08-03 被 v0.2 取代）

> 本节由 ingest [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] 后、用户确定"立 bridge-SDE 为方向"（选项 B）时起草，是 thesis.md 的**首个实质版本**。措辞和押注力度请你自己校准。

> 🔁 **2026-06-18 方向复审中**：师兄推 energy-guided conditional generation（EGSDE 路线）作为候选新方向，已记入 [[research/ideas]] 顶部 Active 条。bridge-SDE 这条线本身已"无存活方法级 idea"（见 ideas Killed），故本 thesis 方向**暂挂复审**——energy-guidance 须先过 3-sweep 纪律才考虑替换下方 v0.1。FlowCycle 引用已按用户指示移除（RF-Inversion 接管"ODE 侧已成熟"论据）。

**研究问题**：把 text-guided image editing 放进"两端分布之间 transport"的统一视角下，**系统性发展 bridge-SDE 一侧的 inversion / editing 理论与方法**——这正是 bridge-ODE 侧（[[wiki/methods/rectified-flow|RF]]、RF-Inversion）已经成熟、而 bridge-SDE 侧（[[wiki/methods/ddbm|DDBM]]、I²SB、Schrödinger Bridge）明显欠发达的缺口。

**核心论点（草稿）**：
1. 生成 / 编辑 / 翻译可统一为"两个端点分布之间、由学到的 drift/score 驱动、迭代采样的 transport"；denoising（噪声端点）只是特例。组织轴见 [[wiki/overview]] 推论 4 与 [[wiki/concepts/diffusion-bridge]]。
2. 该 transport 有 **SDE（随机）** 与 **ODE（确定）** 两种 realization。ODE 侧的 inversion 往返（[[wiki/concepts/probability-flow-ode|PF-ODE]] 可逆性）已被 RF-Inversion 等利用；SDE 侧因随机项，inversion 闭合困难、基本空白——**这是本 thesis 的施力点**。
3. 预期贡献：bridge-SDE 的 inversion / editing 工具，或 SDE↔ODE spectrum 上 inversion 稳定性的统一刻画。

**诚实边界（写进 thesis 是为了不自欺）**：
- "bridge 可 SDE/ODE 双构造"的 lens 源自 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo 2023），**非本工作原创**；本工作的 novelty 必须落在 SDE 侧的具体 inversion/方法贡献。
- DDBM 是 **translation** 论文，不是 text-editing；"editing = paired transport"目前是**泛化假设，尚未验证**。
- 候选 idea 与 prior-art sweep 清单见 [[research/ideas]]（2026-06-01 条）。

## 历史版本（diff）

<!-- 每次 thesis 更新前的版本归档于此，便于回看。 -->

- v0（2026-05-05 → 2026-06-01）：空占位"待 ingest 后定稿"。
- v0.1（2026-06-01 → 2026-08-03）：bridge-SDE inversion/editing；prior-art sweep 后方向复审，随后被 reject-and-skip 的实验主线取代。
