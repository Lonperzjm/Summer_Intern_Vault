---
type: method
title: "Learn to Guide"
aliases: [Learned Guidance, Self-Consistency Guidance]
tags: [diffusion, flow-matching, guidance, classifier-free-guidance]
status: active
created: 2026-08-14
updated: 2026-08-14
sources: ["[[wiki/sources/galashovLearnGuideYour2025]]"]
---

# Learn to Guide

## 一句话总结

冻结 conditional/unconditional 生成模型，用跨时间 self-consistency 学习 $\omega_\phi(c,s,t)$，把手调 CFG scale 变成 condition- 与 step-dependent 的轻量控制器。

## 核心对象

$$
\hat x_\theta(x_t,c;\omega)=\hat x_\theta(x_t,c)+\omega_\phi(c,s,t)\big[\hat x_\theta(x_t,c)-\hat x_\theta(x_t,\varnothing)\big].
$$

- 输入：condition embedding、proposal time $t$、target time $s$；输出非负 scalar weight。
- 冻结：conditional/unconditional denoiser 或 velocity model。
- 训练：真实 $(x_0,c)$、直接加噪 target、guided proposal；不需要最优 scale 标签。

## Self-consistency objective

匹配 $x_0\rightarrow x_s$ 与 $x_0\rightarrow x_t\rightarrow\tilde x_s$ 在 $s$ 的分布。主方法使用带粒子排斥项的 energy-MMD；$\ell_2$ 版更便宜但更敏感。训练采用比推理相邻步更大的 time gap。

## 成本

推理仍需 CFG 的 conditional + unconditional 两个 backbone forward，外加轻量 guidance network。它需要离线训练和数据访问，不是 training-free；但不 fine-tune backbone。

## 关系与边界

- [[wiki/concepts/classifier-free-guidance|CFG]]：global scalar → $\omega(c,s,t)$。
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]]：手工矩形窗 → learned continuous schedule。
- [[wiki/methods/cfg-plus-plus|CFG++]]：前者学习“多强”，后者改变“进入更新的哪部分”；可组合但尚无联合实验。
- [[wiki/concepts/flow-matching|Flow Matching]]：在 1.05B velocity model 上有直接实验。
- 控制器不读取 $x_t$，不是 state-adaptive policy；self-consistency 是比 marginal consistency 更强的实用 surrogate。

## 出处

- [[wiki/sources/galashovLearnGuideYour2025]]
