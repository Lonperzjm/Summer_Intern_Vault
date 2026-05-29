---
type: concept
title: Noising Strength（加噪强度 $t_0$ / img2img strength）
aliases: [noising strength, 加噪强度, t0, "img2img strength", "SDEdit t0", "denoising strength"]
tags: [diffusion, editing, score-sde, fidelity-controllability, hyperparameter]
status: stable
created: 2026-05-29
updated: 2026-05-29
sources: ["[[wiki/sources/mengSDEditGuidedImage2022]]"]
---

# Noising Strength（加噪强度 $t_0$）

## 一句话定义

在 **noising-based 编辑**（[[wiki/methods/sdedit|SDEdit]]、SD img2img）里，把输入图加噪到的**中间时刻 $t_0$**（等价地：加多少噪 $\sigma(t_0)$）。它是 **realism↔faithfulness（更一般地 fidelity↔controllability）权衡的单一旋钮**：$t_0$ 越大越真实但越不忠于输入，越小越忠实但越可能保留输入的伪影。SD img2img 的 `strength` 参数即 $t_0$ 的归一化版。

## 数学/技术细节

给定输入 guide $x^{(g)}$、归一化时间 $t\in[0,1]$（$t{=}0$ 数据、$t{=}1$ 纯噪声，按 diffusion 约定）：

$$
x^{(g)}(t_0) = \begin{cases}
x^{(g)} + \sigma(t_0)\,z, & \text{VE-SDE}\\[4pt]
\alpha(t_0)\,x^{(g)} + \sigma(t_0)\,\epsilon, & \text{VP / DDPM}
\end{cases}\qquad z,\epsilon\sim\mathcal N(0,I),
$$

随后从 $t_0$ 跑 reverse 到 $0$。**反向只走 $t_0\to 0$ 这一段**（不是从 $1$），所以：

- **去噪步数也随 $t_0$ 缩放**：SD img2img 实际步数 ≈ `strength × steps`（`strength=0.5`、`steps=50` → 实跑约 25 步）。
- $t_0$ 同时编码两件事：**从哪个 $t$ 开始 reverse**（介入时间步）+ **加多少噪**（介入强度）——在 noising-based 编辑里这两个旋钮**耦合成一个**。

### realism↔faithfulness 单调曲线（[[wiki/sources/mengSDEditGuidedImage2022|SDEdit]] Fig 3）

| $t_0$ | realism | faithfulness | 行为 |
|---|---|---|---|
| $\to 1$（强加噪） | 高 | 低 | 接近无条件生成，guide 几乎被忽略 |
| 中（≈0.3–0.6） | 平衡 | 平衡 | 多数任务最佳工作区 |
| $\to 0$（弱加噪） | 低 | 高 | guide 伪影保留，不够真实 |

🟡 论文 p.4：「increased realism but decreased faithfulness as $t_0$ increases」——这是本旋钮的原始经验出处。

### "分布重合"几何（SDEdit Fig 1）

$t_0$ 的合理范围由一个几何前提决定：guide 加噪分布 $p_{t_0}(\cdot\mid x^{(g)})$ 必须与目标自然图加噪分布**有大重合**。$t_0$ 太小则两分布分离、reverse 拉向 guide 的"非真实"邻域；$t_0$ 太大则 guide 结构信息被噪声淹没。最优 $t_0^*$ 落在"刚好重合"处——如何由两分布的 KL / Wasserstein 预测 $t_0^*$ 是 open question。

## 与其他概念的关系

- **vs [[wiki/concepts/classifier-free-guidance|CFG]] 的 guidance strength $w$**：**不同旋钮**。$t_0$ 调"从哪个 $t$ 开始 + 加多少噪"（一次性、决定起点）；$w$ 调"每个 $t$ 上 conditional/unconditional 外推多少"（逐 $t$、决定方向放大）。两者可组合（SDEdit + CFG），是否正交是 open question。
- **vs [[wiki/methods/controlnet|ControlNet]] 的 sideband**：ControlNet 在**所有 $t$** 注入条件残差（无 $t_0$ 旋钮）；noising strength 是**单次 $t_0$** 决定。这是「全 $t$ 介入 vs 单次介入」对照（见 [[wiki/overview]] 推论 2 caveat）。
- **是 [[wiki/overview]] 推论 2 的核心量**：推论 2「高 $t$ 改结构、低 $t$ 改细节」中那个"在哪个 $t$ 介入"的旋钮，在 noising-based 编辑里就**具体化为 $t_0$**——把抽象假设变成可测超参。
- **与 [[wiki/concepts/diffusion-process|前向加噪]] / [[wiki/concepts/score-sde|Score SDE]]**：$\sigma(t_0)$ / $\alpha(t_0)$ 直接来自前向过程的 noise schedule。

## 在 text-guided editing 中的作用

- **img2img 的灵魂参数**：SD / SDXL / 几乎所有 img2img pipeline 都暴露 `strength`（或 `denoising_strength`）——它就是 $t_0$。用户调它在"重画"与"微调"间滑动。
- **全局旋钮的局限**：$t_0$ 是**整图统一**的——无法对"要改的区域用大 $t_0$、要保的区域用小 $t_0$"。这是 noising-based 编辑的天花板，催生了 mask-based（局部 $t_0$）、attention-based（局部条件）等更精细派系。
- **thesis 落点**：fidelity↔controllability trade-off 的**最早可测实例**；与 inversion-based（DDIM-inversion / Null-text）构成"忠实度 vs 成本"光谱的零成本端点。

## 出处与引用

- 主要出处：[[wiki/sources/mengSDEditGuidedImage2022]] §4（$t_0$ 定义与 Fig 3 trade-off 曲线）
- 理论底座：[[wiki/concepts/score-sde]]（VE/VP 加噪 schedule）
- 工程落地：SD img2img `strength`（Rombach et al. / diffusers 实现）
