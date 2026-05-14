---
type: concept
title: Diffusion Process（forward / reverse 双过程）
aliases: [forward process, reverse process, 加噪链, 去噪链]
tags: [diffusion, foundational]
status: active
created: 2026-05-10
updated: 2026-05-14
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]"]
---

# Diffusion Process

## 一句话定义

由一条**固定的、把数据逐步加噪到纯高斯**的 Markov 链（forward）与一条**学习得到的、逐步去噪还原数据**的 Markov 链（reverse）组成；二者都被参数化为高斯转移。

## 数学/技术细节

### Forward（固定）

$$q(x_t\mid x_{t-1}) = \mathcal{N}(x_t;\, \sqrt{1-\beta_t}\,x_{t-1},\, \beta_t I)$$

由 Markov 性可证一步闭式：

$$q(x_t\mid x_0) = \mathcal{N}(x_t;\, \sqrt{\bar\alpha_t}\,x_0,\, (1-\bar\alpha_t)I),\quad \bar\alpha_t = \prod_{s=1}^{t}(1-\beta_s)$$

DDPM 取 $T=1000$，$\beta_t$ 线性从 $10^{-4}$ 到 $0.02$。

### 反向真后验（也是高斯，闭式）

$$q(x_{t-1}\mid x_t,x_0) = \mathcal{N}\!\left(x_{t-1};\ \tilde\mu_t(x_t,x_0),\ \tilde\beta_t I\right)$$

$$\tilde\mu_t = \frac{\sqrt{\bar\alpha_{t-1}}\beta_t}{1-\bar\alpha_t}x_0 + \frac{\sqrt{\alpha_t}(1-\bar\alpha_{t-1})}{1-\bar\alpha_t}x_t,\qquad \tilde\beta_t = \frac{1-\bar\alpha_{t-1}}{1-\bar\alpha_t}\beta_t$$

### Reverse（学习）

$$p_\theta(x_{t-1}\mid x_t) = \mathcal{N}(x_{t-1};\,\mu_\theta(x_t,t),\,\Sigma_\theta(x_t,t))$$

当 $\beta_t$ 足够小时，反向真过程也（近似）是 Gaussian——所以选 Gaussian 形式不会损失表达力（**仅在小步长极限下严格**，见下方"局限"）。

## 与其他概念的关系

- 训练目标见 [[wiki/concepts/variational-bound-elbo]]
- 核心参数化（预测 ε 而非 μ）见 [[wiki/concepts/epsilon-parameterization]]
- 与 score-based 视角的等价见 [[wiki/concepts/score-matching]] / [[wiki/concepts/langevin-dynamics]]
- forward 闭式采样依赖 [[wiki/concepts/reparameterization-trick]]

## 在 text-guided editing 中的作用

- 几乎所有编辑方法都把"**编辑**"建模为：(a) 把图像 invert 回 forward 链上的某个 $x_t$；(b) 在 reverse 链中以条件信号引导得到目标。理解 forward/reverse 的几何（noise schedule、$\bar\alpha_t$ 曲线）是判断编辑保真度的基础。

## 局限 / Open questions

- "$\beta_t$ 小 ⇒ reverse 也是 Gaussian"在离散链上是渐近论；连续时间极限（SDE 视角）由 [[wiki/concepts/score-sde|Score SDE]] 严格化，但离散链 $T=1000$ 是否"足够小"是经验判断
- 待补：连续时间 SDE 极限小节，引用 [[wiki/concepts/score-sde|Song et al. 2021]]

## 出处与引用

- [[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]（forward/reverse 闭式、$\tilde\mu_t,\tilde\beta_t$）
- 上游：[[wiki/methods/diffusion-2015|Sohl-Dickstein et al. 2015]]
