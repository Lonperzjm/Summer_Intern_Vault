---
type: research
title: 论文 Thesis 演化
status: draft
created: 2026-05-05
updated: 2026-06-01
---

# Thesis

<!-- 我的论文核心论点。在阅读中演化，由 Claude Code 协助维护，每次改动必须经过我确认。 -->

## 当前版本（v0.1 草稿 · 2026-06-01 · ⚠️ 待用户精修）

> 本节由 ingest [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] 后、用户确定"立 bridge-SDE 为方向"（选项 B）时起草，是 thesis.md 的**首个实质版本**。措辞和押注力度请你自己校准。

**研究问题**：把 text-guided image editing 放进"两端分布之间 transport"的统一视角下，**系统性发展 bridge-SDE 一侧的 inversion / editing 理论与方法**——这正是 bridge-ODE 侧（[[wiki/methods/rectified-flow|RF]]、RF-Inversion、我的 FlowCycle）已经成熟、而 bridge-SDE 侧（[[wiki/methods/ddbm|DDBM]]、I²SB、Schrödinger Bridge）明显欠发达的缺口。

**核心论点（草稿）**：
1. 生成 / 编辑 / 翻译可统一为"两个端点分布之间、由学到的 drift/score 驱动、迭代采样的 transport"；denoising（噪声端点）只是特例。组织轴见 [[wiki/overview]] 推论 4 与 [[wiki/concepts/diffusion-bridge]]。
2. 该 transport 有 **SDE（随机）** 与 **ODE（确定）** 两种 realization。ODE 侧的 inversion 往返（[[wiki/concepts/probability-flow-ode|PF-ODE]] 可逆性）已被 FlowCycle 等利用；SDE 侧因随机项，inversion 闭合困难、基本空白——**这是本 thesis 的施力点**。
3. 预期贡献：bridge-SDE 的 inversion / editing 工具，或 SDE↔ODE spectrum 上 inversion 稳定性的统一刻画。

**诚实边界（写进 thesis 是为了不自欺）**：
- "bridge 可 SDE/ODE 双构造"的 lens 源自 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]（Albergo 2023），**非本工作原创**；本工作的 novelty 必须落在 SDE 侧的具体 inversion/方法贡献。
- DDBM 是 **translation** 论文，不是 text-editing；"editing = paired transport"目前是**泛化假设，尚未验证**。
- 候选 idea 与 prior-art sweep 清单见 [[research/ideas]]（2026-06-01 条）。

## 历史版本（diff）

<!-- 每次 thesis 更新前的版本归档于此，便于回看。 -->

- v0（2026-05-05 → 2026-06-01）：空占位"待 ingest 后定稿"。
