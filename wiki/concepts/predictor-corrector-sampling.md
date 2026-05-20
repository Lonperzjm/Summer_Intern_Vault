---
type: concept
title: Predictor-Corrector 采样（PC sampler）
aliases: [predictor-corrector, PC sampler, PC 采样]
tags: [diffusion, sde, sampling, score-based]
status: active
created: 2026-05-20
updated: 2026-05-20
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]"]
---

# Predictor-Corrector 采样

## 一句话定义

把"数值 SDE 求解器（predictor）"与"基于 score 的 MCMC（corrector）"交替组合的采样框架：predictor 沿反向时间推进一步，corrector 在**当前时间步**原地跑若干步 [[wiki/concepts/langevin-dynamics|Langevin MCMC]]，把样本拉回该时刻应有的分布 $p_t$。

## 为什么需要 corrector

纯反向 SDE 离散化（predictor-only）会累积两类误差：① 离散步长带来的偏差；② 随机项 $\mathrm d\bar w$ 偶尔把样本"踢"离高概率区。直觉（采纳用户 takeaway #5）：我们希望样本最终坍缩到数据分布的尖峰附近，但单纯反向走可能因离散/随机偏差跑偏。corrector 利用我们**手上已有的 score** $\nabla_x\log p_t(x)$ 把样本重新校准到尖峰。

关键性质：corrector 的 Langevin 稳态分布**就是** $p_t$，对应 [[wiki/concepts/fokker-planck-equation|Fokker-Planck]] 的 $\partial_t p_t=0$——所以它**不改变时间分布、不推进时间**，只在固定 $t$ 上提纯，不会扰乱 predictor 建立的时间进度。

## 流程

```
给定: 训练好的 score s_θ(x, t); 反向时间网格 t_N > ... > t_0
x ~ p_T  (先验, 如 N(0, σ²I) 或 N(0, I))
for i = N, N-1, ..., 1:
    # Predictor: 反向 SDE 离散一步 (如 reverse diffusion / ancestral)
    x = predictor_step(x, t_i -> t_{i-1}, s_θ)
    # Corrector: 在 t_{i-1} 原地跑 m 步 Langevin
    for j = 1..m:
        z ~ N(0, I)
        x = x + ε · s_θ(x, t_{i-1}) + sqrt(2ε) · z
return x
```

corrector 单步即 Langevin：$x \leftarrow x + \varepsilon\,\nabla_x\log p_t(x) + \sqrt{2\varepsilon}\,z$（对应 SDE $\mathrm dx=\nabla_x\log p_t(x)\,\mathrm d\tau+\sqrt2\,\mathrm dw_\tau$）。

## 实验结论

- PC 通常**优于把 predictor 步数翻倍但不加 corrector**（如 P2000 基线）：相同算力下 corrector 更划算。
- step size $\varepsilon$ 常按 signal-to-noise ratio 自适应设定；步数 $m$ 是质量/算力旋钮。

## 与其他概念的关系

- corrector = [[wiki/concepts/langevin-dynamics|annealed Langevin]] 的"单时间步"复用（[[wiki/methods/ncsn|NCSN]] 的采样精神被吸收为 corrector）
- 与 [[wiki/concepts/probability-flow-ode|probability-flow ODE]] 并列，是 [[wiki/concepts/score-sde|Score SDE]] 的随机性采样器
- predictor 步即反向 SDE 的离散化（[[wiki/concepts/diffusion-process]] reverse 链的连续版）

## 在 text-guided editing 中的作用

- corrector 的 Langevin 步可被注入条件 score，作为 inversion / 编辑的精炼步（"在条件 score 场中走 K 步"）

## 出处与引用

- [[wiki/sources/songScoreBasedGenerativeModeling2021]]（PC 框架提出）
- Langevin/HMC 基础：Parisi 1981、Neal et al. 2011
