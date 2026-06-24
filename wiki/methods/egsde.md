---
type: method
title: EGSDE（Energy-Guided SDE）
aliases: [EGSDE, Energy-Guided SDE, energy-guided stochastic differential equations]
tags: [diffusion, score-sde, guidance, energy-guidance, unpaired-i2i, translation, training-free-generator]
status: stable
created: 2026-06-21
updated: 2026-06-21
sources: ["[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]"]
family: guidance
---

# EGSDE（Energy-Guided SDE）

> family 标 `guidance`：EGSDE 不重训生成器，靠**采样期能量梯度**把条件注入冻结的目标域 [[wiki/concepts/score-sde|score SDE]]。是 [[wiki/concepts/energy-guidance|energy guidance]] 概念的代表方法、[[wiki/concepts/classifier-guidance|classifier guidance]] 的能量化推广。

## 一句话总结

**冻结目标域 SDE + 双域 energy 引导**：反向采样时把引导项 $-\nabla_{y_t}\mathcal E(y_t,x_0,t)$ 加到目标域 score 上，energy 由"域特定专家（降源域相似度 → realism）+ 域无关专家（保结构 → faithfulness）"组成，做 unpaired I2I。

## 核心机制

| 组件 | 内容 |
|---|---|
| 底座 | 任意预训练**目标域** [[wiki/concepts/score-sde\|score SDE]]（冻结，不重训） |
| 输入 | 源图 $x_0$ → 加噪到 $M=0.5T$ 当反向起点 $y_M$ |
| 引导 | $s_{\text{EGSDE}}=s_{\mathcal Y}(y_t,t)-\nabla_{y_t}\mathcal E(y_t,x_0,t)$ |
| 能量 | $\mathcal E=\lambda_s S_s-\lambda_i S_i$（$S_s$ 域特定余弦相似度 / $S_i$ 低通负 L2） |
| 专家 1 $E_s$ | 源/目标域分类器除末层的特征，**时间相关 noise-aware**（须训） |
| 专家 2 $E_i$ | 低通滤波器（无需训） |
| 旋钮 | $\lambda_s$↑ realism / $\lambda_i$↑ faithfulness |
| 训练 | 生成器**无**；energy 侧只需训域分类器 $E_s$ |

## Pipeline

```
Require: 源图 x0, 目标域 score s_Y, 域分类器特征 E_s, 低通 E_i, λ_s, λ_i, M=0.5T
y ← 加噪(x0, M)                      # 起点 = 噪化源图
for t = M down to 0:
    x_t ← 加噪(x0, t)               # ★ 把源图噪化到同一 t（noisy-aligned）
    S_s ← cos_sim(E_s(y,t), E_s(x_t,t))      # 域特定：降它 → realism
    S_i ← -||E_i(y,t) - E_i(x_t,t)||^2        # 域无关：升它（拉近）→ faithfulness
    grad ← λ_s ∇_y S_s - λ_i ∇_y S_i          # = ∇_y E
    s_guided ← s_Y(y,t) - grad
    y ← reverse_SDE_step(y, s_guided, t)
return y
```

> ★ 关键：第 3 行把**源图**噪化到当前 $t$ 做 noisy-to-noisy 比较，**不**从 $y$ 估 $\hat y_0$——这是 EGSDE 的 noisy-aligned 设计（见 [[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]] Method §3）。

## 适用场景与限制

**适用**：unpaired I2I 翻译（Cat→Dog、Wild→Dog、Male→Female）；任意"有目标域 score model + 可定义双域 energy"的条件生成任务。

**限制**：
- **faithful 专家弱**：只是低通滤波器，难表达复杂结构约束。
- **$E_s$ 须 noise-aware 重训**：energy 作用在 noisy state、且每个 task/域对要重训域分类器——不能即插现成 clean 判别器。
- **随机 SDE**：同源图多次运行结果不同（多样性的另一面是不可复现）。

## 关联

- 出处：[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]
- 概念：[[wiki/concepts/energy-guidance]]（本方法的抽象）；[[wiki/concepts/classifier-guidance]]（被推广的原型）；[[wiki/concepts/score-sde|Score SDE]]（反向 SDE 底座）
- 基线 / 同族：[[wiki/methods/sdedit|SDEdit]]（$p_{r1}$ 专家即 SDEdit；EGSDE = SDEdit + energy）；ILVR、CycleGAN 系（待 ingest）
- 条件注入对照：[[wiki/methods/controlnet|ControlNet]]（改网络）/ [[wiki/methods/ddbm|DDBM]]（改端点）/ EGSDE（采样期能量引导）
- clean-estimate 对照：[[wiki/methods/freedom|FreeDoM]] —— 同是能量引导，但用 $\hat x_0$ 点估计 + 现成模型（免重训），与 EGSDE 的 noisy-aligned 正相反（见 [[wiki/concepts/conditional-diffusion]] §5）
- 下游方向（[[research/ideas]]）：clean-estimate 能量 $E(\hat y_0,x_0)$ + 搬到 [[wiki/methods/rectified-flow|RF]] / [[wiki/concepts/flow-matching|FM]]
- 作者：[[wiki/entities/jun-zhu]]、Fan Bao、Chongxuan Li、Min Zhao；[[wiki/entities/tsinghua-university|Tsinghua]]
