---
type: research-note
title: 路径交叉区域的 Reject-and-Skip 采样直觉
aliases: [奇异点统一框架, Reject-and-Skip]
tags: [flow-matching, singularity, adaptive-step, reject-and-skip, original-analysis]
status: active
created: 2026-07-28
updated: 2026-08-02
triggered_by: ["[[wiki/sources/chaTrainingFreeRefinementFlow2026]]", "[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]"]
---

# 路径交叉区域的 Reject-and-Skip 采样直觉

> 原始想法见 [[raw/literature-notes/chaTrainingFreeRefinementFlow2026plus]]。本文只整理原始手记中的机制与两种应对策略；不把后续延伸出的假设当成既定结论。

## 原始观察

考虑两组靠近或交叉的 Flow Matching 插值路径。在高歧义位置，marginal velocity field 可能给出多个条件速度的平均值。由于确定性 ODE 轨迹不能相交，实际学到的轨迹会在该区域弯曲、分流。

![[chaTrainingFreeRefinementFlow2026plus-1785231261637.webp]]

这里暂时用两个量描述局部现象：

- $l$：速度平均明显发生的局部区域尺度。它只是待实验定义的操作性量，可能同时受 conditional velocity ambiguity、模型容量和训练误差影响；目前不能简单等同于“模型空间分辨率”。
- $\Delta=\|v\|\delta t$：一次离散积分造成的空间位移。

原始手记提出的风险是：当离散步长与该局部区域尺度不匹配时，一次 trial step 可能落入速度平均严重的区域；下一次模型调用得到不属于任一真实分支的平均速度，从而把样本推向低密度/OOD 区域。

![[raw/assets/chaTrainingFreeRefinementFlow2026plus-1785241803614.webp]]

这是一个**待验证机制假设**。$d_{\min}<\Delta$ 只能作为候选几何条件，不能单独推出一定命中异常区；命中概率、$l$ 的定义及其与 $\Delta$ 的关系都需要实验测量。

## 原始的两种应对策略

### 1. 拒绝后缩步（reject-and-shrink）

先试探一步。如果新位置的速度相对旧位置发生异常变化，则拒绝该 trial step、回滚到上一个安全状态，并减小步长，细致解析弯曲区域。

![[chaTrainingFreeRefinementFlow2026plus-1785241855762.webp]]

这与 adaptive step-size solver 的基本行为相近，可作为主要 baseline。

### 2. 拒绝后跨步（reject-and-skip）

同样先试探一步并检测速度突变；若判定试探点落入局部不可信区域，则拒绝并回滚，但不缩小步长，而是从上一个安全状态主动增大步长，尝试一次越过该区域。跨越后再恢复正常积分。

![[chaTrainingFreeRefinementFlow2026plus-1785241874885.webp]]

核心不是“始终使用更大的步长”，而是：

> **trial → detect → reject/rollback → skip → validate/correct**

跨步候选不能无条件接受。需要验证出口已经恢复到速度变化较平稳的区域；验证失败时应继续搜索合适的跨步长度，或回退到缩步策略。跨步后还可以增加 corrector，以减少大步外推造成的偏差。

## 一个待实现的算法骨架

给定安全状态 $(x_k,t_k)$ 和候选步长 $h$：

1. 用当前速度 $v_k=v(x_k,t_k)$ 产生 trial state $x_{\mathrm{trial}}$。
2. 在 trial endpoint 计算 $v_{\mathrm{trial}}$。
3. 用归一化速度变化、embedded Euler--Heun defect、divergence 或其组合判断 trial 是否异常。
4. 若正常，接受该步，并复用 $v_{\mathrm{trial}}$。
5. 若异常，拒绝并回滚到 $(x_k,t_k)$：
   - shrink 分支：减小 $h$ 后重试；
   - skip 分支：增大 $h$，寻找异常区之后的候选出口。
6. skip 出口通过一致性检查后，可做 corrector；否则回退到 shrink。

候选检测量之一：

$$
D(h)=\frac{\|v(x_{\mathrm{trial}},t_k+h)-v(x_k,t_k)\|}{\|v(x_k,t_k)\|+\varepsilon}.
$$

$D(h)$ 只是最简单的起点，不预设它足以区分“真实高曲率”与“模型不可信区域”。

## 与 FDS 的关系：暂不宣称正交

FDS 在 state space 中寻找 divergence 更低的邻近位置；这里的原始想法是在一次失败 trial 后改变时间离散策略。两者在操作上分别修改 state 和 step，但是否解决同一失败机制、是否真正正交、能否组合，都仍需实验和更严格分析。当前不把“三种策略正交”写成结论。

## 需要优先回答的问题

- [ ] 什么信号能可靠地区分正常高曲率区域与不可信的速度平均区域？
- [ ] trial step 的异常应如何定义：速度变化、Euler--Heun defect、divergence，还是联合判据？
- [ ] reject-and-skip 如何选择跨步长度并确认已经离开异常区？
- [ ] 大步外推如何保持正确分支？corrector 或多候选出口能否降低跳错分支的概率？
- [ ] skip 何时优于 shrink，何时必须回退？
- [ ] $l$ 如何操作性定义和测量，而不把 Bayes ambiguity 与有限模型误差混在一起？
- [ ] 在解析 2D toy 中，skip 是否降低低密度/OOD 率，而不仅是改变轨迹误差？

## 实验顺序

1. 解析或高精度可求解的 2D 双模态 toy：直接观察 trial、rollback、shrink 与 skip。
2. 学习得到的 2D velocity MLP：研究模型容量和局部平均区域。
3. CIFAR-10 Flow Matching backbone：记录轨迹上的速度变化、local defect、divergence 与长尾失败。
4. 只有机制在前三层成立后，再迁移到大型预训练 backbone。

## 前序

← [[research/notes/2026-07-27-high-dim-crossing-probability|高维交叉概率分析]]
