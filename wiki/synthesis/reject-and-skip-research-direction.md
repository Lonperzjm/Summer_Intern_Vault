---
type: synthesis
title: Reject-and-Skip 研究方向：从条件速度歧义到推理时 Coupling 干预
aliases: [Reject-and-Skip Research Direction, 推理时 Coupling 干预]
tags: [flow-matching, rectified-flow, solver, transport-coupling, research-direction]
status: active
created: 2026-08-03
updated: 2026-08-03
sources: []
evidence: ["[[research/experiments/2026-08-02-reject-and-skip-toy-report]]", "[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]"]
---

# Reject-and-Skip 研究方向：从条件速度歧义到推理时 Coupling 干预

## 研究问题

当 [[wiki/concepts/flow-matching|Flow Matching]] 的 marginal velocity 因多分支条件速度平均而在局部区域对单条轨迹不可信时，能否在不重训 backbone 的前提下检测该区域，回滚并跨越它，以结构化离散偏置选择更有利的 [[wiki/concepts/transport-coupling|transport coupling]]，并在固定 NFE 下改善最终生成分布？

## 当前证据链

| 层级 | 已有证据 | 能说明什么 | 不能说明什么 |
|---|---|---|---|
| 解析 2D toy | Oracle ambiguity、真实密度、branch label 均可得；skip 降低粗 Euler outlier，同时显著偏离 RK4 | 跨越局部平均坏场可以得到另一条合理 coupling | 神经网络中是否有可观测的同类异常 |
| 官方 CIFAR-10 1-RF | 稳定 backbone、固定 solver、trajectory detector、endpoint ablation、adaptive Heun 已完成 | 真实速度场数值难度不均匀，低成本 trial 信号有预测力，shrink 会集中消耗预算 | 内部困难是否来自 conditional ambiguity；skip 是否改善生成分布 |
| 尚缺机制实验 | velocity-return scan、出口可信度、完整 rollback solver、等 NFE 分布评测 | — | 方向能否成为真实模型方法贡献 |

## 核心方法学转向

早期的错误判据是把“大步是否更接近 RK4 / 两个小 Heun 步”当成 skip 成败。Toy 已表明 skip 可能有意偏离 marginal ODE 而保持合理 endpoint distribution，因此必须分开三个目标：

1. **ODE accuracy**：是否接近高精度 marginal ODE；
2. **distribution quality**：endpoint 是否匹配目标分布、是否减少 OOD/重尾失败；
3. **coupling behavior**：样本级输入—输出配对如何被改变。

本方向以第 2 项为主目标，第 3 项解释机制，第 1 项只记录代价与偏离。

## 必须排除的伪解释

- **Endpoint pathology**：官方 1-RF 的最强异常贴近 $t=1$；若方法只是在避免终点求值，explicit midpoint 已是更简单的基线。
- **普通高阶法优势**：大步 RK4 对正常事件也更强，不能当 candidate-specific skip 证据。
- **Adaptive solver 偶然跨区**：必须证明显式 detector、rollback 和 exit validation 比普通 controller 更稳定。
- **时间表泄漏**：detector 必须具有同一时间片内的样本区分力，并在独立种子上校准。
- **Reference 指标错位**：更大 RK4 RMSE 只能说明 coupling change，不能推出 FID/OOD 变差。

## 下一阶段最小实验闭环

1. 在固定高精度入口上扫描 $m\in\{1,2,3,4,6,8\}$，记录 relative velocity change 与 angle 的 return curve。
2. 用独立随机种子校准入口触发阈值、return 阈值和最大跨度。
3. 组合 projected endpoint consistency、局部扰动敏感度或 MC-dropout variance，形成出口验证；避免只用 embedded defect。
4. 实现 `trial → reject/rollback → scan → accept/correct → fallback`，逐样本记录真实 NFE、触发、失败回退与跨度。
5. 先做 1k 视觉与失败型检查，再做 5k、3 seeds 的 KID、近似 FID、precision/recall、feature OOD；只有稳定改善才进入 50k。

## 必须击败的基线

- 固定 explicit midpoint；
- 固定 RK4；
- adaptive Heun / step-doubling；
- shrink-only；
- 无 rollback 的大步；
- 若成本允许，和 [[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]] 这类 spatial correction 对照或组合。

## 当前判断

这是一个**存活但尚未完成机制迁移**的研究方向。Toy 证明了“主动改变 coupling 可能有益”的存在性；1-RF 证明了真实模型上的诊断基础和强 baseline 已具备。论文级贡献仍取决于：非 oracle detector 是否对齐状态歧义、velocity return 是否真实存在，以及完整方法能否在严格等 NFE 下稳定改善生成分布。

## 关联页面

- 方法：[[wiki/concepts/reject-and-skip]]
- 数值坐标：[[wiki/concepts/ode-solver-taxonomy]]
- Coupling：[[wiki/concepts/transport-coupling]]
- 实验证据：[[research/experiments/2026-08-02-reject-and-skip-toy-report]]、[[research/experiments/2026-08-03-official-1rf-solver-diagnostics]]
