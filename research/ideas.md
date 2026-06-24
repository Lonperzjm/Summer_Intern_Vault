---
type: research
title: Idea 池
status: active
created: 2026-05-05
updated: 2026-06-24
---

# Ideas

<!-- 候选 idea 暂存。每条注明触发源（[[sources/...]]）与日期。 -->

## Active

### [2026-06-18 · 更新 2026-06-24] Energy-guided conditional generation —— sliver 已定（待第一个实验）

来源：师兄 6/2 推 [EGSDE (NeurIPS'22)](https://arxiv.org/abs/2207.06635)（可迁移范式 **discriminative logits/reward → energy → guidance**）；6/23 进一步把 [[wiki/methods/freedom|FreeDoM]] 定为 baseline 并给出三段改进框架。已 ingest：[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022|EGSDE]]、[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b|FreeDoM]]。概念骨架：[[wiki/concepts/conditional-diffusion]]、[[wiki/concepts/energy-guidance]]、[[wiki/concepts/training-free-guidance]]。

**核心 novelty 假设**：怎么从 $x_t$ **高效且准确**地拿到对应 $x_0$ 的评分（判别 logits/reward），再用 energy 当桥转成引导。

**✅ sweep 已做（2026-06-24，结论：generic 形式全红）**——比 bridge-SDE 还挤：
- 点估计 $\hat x_0$ + 现成 reward（diffusion）→ DPS / [[wiki/methods/freedom|FreeDoM]] / UGD / LGD / MPGD，被 [TFG 2403.12404](https://arxiv.org/abs/2403.12404) 统一收编。
- energy 搬到 flow → [Energy-Weighted FM (ICLR'25)](https://arxiv.org/pdf/2503.04975)、Energy-Guided FM、[TFG-Flow](https://arxiv.org/pdf/2501.14216)。
- training-free flow 编辑 → [FlowChef (ICCV'25)](https://github.com/FlowChef/flowchef)、[OC-Flow (ICLR'25)](https://arxiv.org/html/2410.18070v2)、D-Flow。
- energy 编辑（EGSDE 后续）→ [DragonDiffusion](https://arxiv.org/pdf/2307.02421)、[Contrastive Energy Prediction (Lu'23)](https://arxiv.org/pdf/2304.12824)。
- **不 KILL 的唯一理由**：坐在师兄/Long 组 flow 在研线上（红海生存三铁律里命中"导师在研线"）。

**🎯 sliver（师兄 6/23 三段框架）**——FreeDoM 三段都还是 heuristic，逐段找原理化改进：

| 段 | FreeDoM 的 heuristic | 原理化方向 | 占用 |
|---|---|---|---|
| ① $x_t\to\hat x_0$ 估计 | Tweedie 单点 | RF 的 $\hat x_0=x_t-t\,v$ 更准、偏差更小 | 较空、正交 |
| ② energy 获取 | 单个现成距离 + 简单加权 | **结构化 E**：保/丢拆分（EGSDE 思路）但用现成模型拼、不重训 | 半占（核心 sliver） |
| ③ energy→guidance | 手调 $\rho_t$ + 欧氏梯度 + 反传 | 原理化步长（接 $g(t)^2$ 尺度）/ 流形投影 / 便宜雅可比 | MPGD/TFG 啃过，flow 上较空 |

**核心收束（最值钱的一句）**：三段的原理化方向**全指向同一个底座 flow/RF**——① $\hat x_0$ 更准、③ 雅可比更便宜、② 结构化 E 在更准的 $\hat x_0$ 上更稳。**这不是三个独立改进，是"换 flow 底座"一个动作同时盘活三段。**

**待正面回答的张力**：clean-estimate 在 flow 上凭什么赢 diffusion？= RF 的 $\hat x_0$ 直、Jensen 偏差更小 → **须实验证伪**。

**下一步**：① 按三段整理思路（已答应师兄，重点 ③ 盘得最少）；② 设计第一个最小实验——选底座（RF/SD3-FLUX）、$F_{\text{keep}}/F_{\text{drop}}$ 用哪俩现成模型、数据集/指标、baseline = EGSDE + FreeDoM。
**待 ingest（🔵）**：DPS、[TFG 2403.12404](https://arxiv.org/abs/2403.12404)（统一框架）、MPGD（③ 的流形改法）。

---

> **[2026-06-01] 当前无存活的"方法级" idea。** bridge/flow/diffusion editing 是 2024–2026 最红的红海，连撞三次（见 Killed）。教训：**不在最热的方法层 armchair 找题**。下一步不是再想一个 carve，而是：
> 1. **更窄的 niche**（具体域/模态/评测空白），离红海远一点；
> 2. **导师 brief**——方向得和 Long 组导师定，把这几周的地图 + 三次撞车结论交回该接的人。
>
> 全景地图：[[wiki/synthesis/bridge-sde-editing-landscape]]。

## Parked

（暂无）

## Killed

### [2026-06-01] "让 SDE bridge 脱离 diffusion 框架"（理论方向）—— KILL（撞 Stochastic Interpolants）

- 想法：把随机桥做成独立对象，直接在两分布间定义、不从 VP/VE 扩散 + Doob h 推。
- 🔴 **撞车**：= [[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants (Albergo 2023)]] 本身（自由选插值/任意两分布/ODE+SDE 可调噪声/还原 SB 全中）。该框架更早更一般，DDBM 反而是它"绑回扩散"的特例。
- 结论：理论 contribution 已满，放弃。直觉正确（"DDBM 依赖扩散是 crutch"），但被占。

### [2026-06-01] "Bridge-SDE editing 全家桶"（含 diversity 旋钮 / 加速 / inversion）—— KILL（三次 sweep 三次撞）

源头 framing（仍有教学价值，但每个落点都被占）：把生成/编辑/翻译统一为"两端分布 transport"，bridge SDE（[[wiki/methods/ddbm|DDBM]]）vs bridge ODE（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]），见 [[wiki/concepts/diffusion-bridge]]。逐个落点的撞车记录：

- **bridge-SDE 加速** → 🔴 [CDBM (NeurIPS'24, 2410.22637)](https://arxiv.org/abs/2410.22637)、[Inverse Bridge Matching Distillation (2502.01362)](https://arxiv.org/abs/2502.01362)。
- **bridge inversion / encoding** → 🔴 [[wiki/methods/dbim|DBIM (ICLR'25)]] 已给确定可逆桥 + booting noise 原语。
- **bridge-ODE inversion/editing** → 🔴 红海：RF-Inversion (ICLR'25)、[RF-Solver-Edit (2411.04746)](https://arxiv.org/abs/2411.04746)、[OT-for-RF-Editing (2508.02363)](https://arxiv.org/abs/2508.02363)。
- **可控多样性 / 随机 flow 编辑（diversity 旋钮）** → 🔴 [OSCAR: Orthogonal Stochastic Control for diversity in Flow Matching (2510.09060)](https://arxiv.org/abs/2510.09060)（training-free 注入正交随机扰动提多样性、不损 fidelity）；Variational Rectified Flow Matching（latent 捕获多模）；Discretized-RF（stochastic velocity 补多样性）；[FlowSlider (2604.02088)](https://arxiv.org/abs/2604.02088)（fidelity-steering 旋钮）。one-to-many 编辑自 Blended Diffusion (2022) 即有。
- **stochasticity / schedule 理论** → 🔴 SB/SI + [Stochasticity Control (2410.21553)](https://arxiv.org/abs/2410.21553)（"最优 coefficient 下 path-KL 与 schedule 无关"）。

**meta 教训**：armchair 想方法 → sweep 查 → 撞，循环三次即证明方法错。红海里出论文靠 execution + 窄 niche + 导师在研线，不靠 blue-sky 新意。
