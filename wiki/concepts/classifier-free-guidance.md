---
type: concept
title: Classifier-Free Guidance (CFG)
aliases: [CFG, classifier-free guidance, 无分类器引导]
tags: [diffusion, guidance, conditioning]
status: stable
created: 2026-05-14
updated: 2026-08-14
sources: ["[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/hoClassifierFreeDiffusionGuidance2022]]", "[[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024]]", "[[wiki/sources/chidambaramWhatDoesGuidance2024]]", "[[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]", "[[wiki/sources/galashovLearnGuideYour2025]]"]
---

# Classifier-Free Guidance (CFG)

> Ho & Salimans, *Classifier-Free Diffusion Guidance*, 2022.

## 一句话定义

不训练单独的分类器，而是**同一个网络同时学条件与无条件 score**（训练时随机丢弃条件 $c$），采样时把两者外推：
$$\tilde\varepsilon_\theta(x_t,t,c) = \varepsilon_\theta(x_t,t,\varnothing) + s\big(\varepsilon_\theta(x_t,t,c) - \varepsilon_\theta(x_t,t,\varnothing)\big)$$
通过 guidance scale $s$（$s=1$ 无引导，Stable Diffusion 常用 $s=7.5$）在样本多样性与条件贴合度之间换挡。

## 数学/技术细节

### 训练：条件 dropout

同一网络 $\varepsilon_\theta(x_t, t, c)$，训练时以概率 $p_\text{uncond}$ 将 $c$ 替换为 $\varnothing$（空文本 embedding）。最佳 dropout 比例约 $p_\text{uncond} \in [0.1, 0.2]$，只需少量 capacity 学 unconditional branch。

### 采样：两次 forward + 外推

每个 denoising step：
1. 条件预测 $\varepsilon_c = \varepsilon_\theta(x_t, t, c)$
2. 无条件预测 $\varepsilon_u = \varepsilon_\theta(x_t, t, \varnothing)$
3. 外推：$\tilde\varepsilon = \varepsilon_u + s(\varepsilon_c - \varepsilon_u)$

等价的 score 写法：$\tilde s = (1+w)\,s_c - w\,s_u$，其中 $s = 1+w$。

### 隐式分类器推导

在 exact score 下，由 Bayes：
$$\nabla_{x_t}\log p(c\mid x_t) = s_c(x_t) - s_u(x_t).$$
于是 CFG 等价于 [[wiki/concepts/classifier-guidance]] 用隐式分类器 $p^{\mathrm{impl}}(c\mid x_t) \propto p(x_t\mid c)/p(x_t)$ 的梯度做 guidance。

### 非保守场 caveat（重要理论细节）

上述等式**仅在 exact score 下成立**。实际 learned scores 是无约束神经网络，其差 $s_\theta(x,c) - s_\theta(x,\varnothing)$ **不一定是保守场**（不一定写成某个标量函数的梯度 $\nabla f$）。因此实际 CFG 并非严格等价于"沿隐式分类器梯度走"——隐式分类器是 motivation 与 exact-case 推导，不是精确描述。这一理论缺口是后续 CFG rescaling、dynamic CFG、guidance distillation 等工作的出发点。

即便各时刻 score 完全准确，也还有另一层 caveat。[[wiki/sources/chidambaramWhatDoesGuidance2024|Chidambaram et al. 2024]] 指出，终点 tilting 与前向 noising 一般不交换：实际 CFG 对 noisy marginal 逐时 tilt，但目标 tilted data distribution 加噪后的 score 通常不同。因此，**逐时 guided score 的代数形式成立，不代表整条 reverse dynamics 最终从朴素 tilted distribution 采样**。

### 几何直觉

$\varepsilon_c - \varepsilon_u$ 是"条件 $c$ 相对于普通数据分布**额外要求的方向**"。CFG 做的是把这个方向放大——是 **extrapolation 而非 interpolation**。$s$ 过大时模型冲过 $\varepsilon_c$，产生过饱和、纹理过强、diversity 崩塌。

[[wiki/sources/chidambaramWhatDoesGuidance2024|What Does Guidance Do?]] 在简单分布上严格化了这个图像：越像竞争类的状态受到越强 correction，最终样本偏向目标类中远离竞争类的边界或尾部。Guidance 更准确地说是**随时间变化的几何 transport bias**，而不是一次静态 density reweighting。

### Large guidance 与 score error 放大

Guidance 主动把轨迹推入低密度 tail，而 $L^2(p_t)$ score error 对这些区域约束很弱。Chidambaram et al. 证明，任意非零 score estimation error 在充分大的 guidance 下都可能导致高概率 off-support。这给过大 scale 后的 distortion / FID 回升提供了理论机制；严格定理基于简单一维混合分布，不能直接当作真实图像模型的普适定量结论。

### 采样代价

每步两次 forward pass，与 sampler 选择（DDPM / DDIM / ODE）正交。

### Guidance interval：从常数 scale 到时间窗

[[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Kynkäänniemi et al. 2024]] 发现全程使用固定 guidance 并非最优：高噪声阶段有害，中间阶段有益，低噪声阶段基本无必要。以 conditional prediction 为基线，可令

$$
w(\sigma)=w\,\mathbf 1[\sigma_{\mathrm{lo}}\le\sigma\le\sigma_{\mathrm{hi}}],
$$

只在中间区间使用 $D_c+w(D_c-D_u)$，区间外退回 $D_c$。这把 CFG 的控制量从单一 guidance scale 扩展为 **scale × interval**，并因区间外无需 unconditional branch 而降低计算量。

机制解释需保持克制：“高噪声时过早收缩分布”“低噪声时条件 correction 已无必要”是与实验一致的直觉；通用最优边界仍依赖模型、sampler 与 noise parameterization。

### CFG++：只 guide denoising，不 guide renoising

[[wiki/methods/cfg-plus-plus|CFG++]] 将 DDIM 的一步拆成 guided clean estimate 与 unconditional renoising：
$$
\hat\epsilon_c^\lambda=\hat\epsilon_u+\lambda(\hat\epsilon_c-\hat\epsilon_u),\qquad
x_{t-1}=\sqrt{\bar\alpha_{t-1}}\hat x_c^\lambda+\sqrt{1-\bar\alpha_{t-1}}\hat\epsilon_u.
$$
主实验用 $\lambda\in[0,1]$ 做 interpolation，而不是用 $\omega>1$ 外推整个更新。Guidance Interval 调“何时 guide”，CFG++ 调“guidance 进入一步更新的哪一部分”。CFG++ 每步仍是标准 CFG 的两次 forward。

### Learn to Guide：从手工 schedule 到条件化学习

[[wiki/methods/learn-to-guide|Learn to Guide]] 冻结原模型，以跨时间 self-consistency 学习 $\omega_\phi(c,s,t)$。它把全局 scale / 手工矩形窗扩展为 condition- 与 step-dependent policy；但不读取 $x_t$，所以不是 state-adaptive。

## Guidance Scale 的效果

$$s\uparrow \;\Rightarrow\; \text{condition fidelity}\uparrow,\quad \text{diversity}\downarrow.$$

ImageNet 128×128 实证（$T=256$）：$w=0$ FID 7.27 / $w=0.3$ FID **2.43**（最优）/ $w=4$ FID 21.53（回升），说明 guidance scale **不是越大越好**。

## 与其他概念的关系

- 作用在 [[wiki/concepts/diffusion-process]] 的 reverse 链上，是 overview「研究杠杆」中的 **guidance** 旋钮
- 前身：[[wiki/concepts/classifier-guidance]]（[[wiki/sources/dhariwalDiffusionModelsBeat2021|Dhariwal & Nichol 2021]]）—— CFG 即"把分类器引导项内化、免去外部分类器"的演化版
- **连续时间理论底座**：[[wiki/concepts/score-sde|Score SDE]]（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]）的条件反向 SDE 由贝叶斯拆分 $\nabla_x\log p_t(x\mid y)=\nabla_x\log p_t(x)+\nabla_x\log p_t(y\mid x)$ 给出——CFG 用条件 dropout 把引导项内化进同一网络
- 推广：[[wiki/concepts/energy-guidance]]（把"隐式分类器"推广为"任意可微能量"）；[[wiki/concepts/training-free-guidance]]（$\hat x_0$ 点估计 + 现成判别器）
- 时间调度：[[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024|Guidance Interval]]（只在中间 noise range 开 CFG，同时改善分布质量并减少双分支计算）
- 动力学解释：[[wiki/sources/chidambaramWhatDoesGuidance2024|What Does Guidance Do?]]（tilting/noising 不交换；boundary/tail concentration；large guidance 放大 tail score error）
- 轨迹修正：[[wiki/methods/cfg-plus-plus|CFG++]]（guided denoising + unconditional renoising）
- 权重学习：[[wiki/methods/learn-to-guide|Learn to Guide]]（冻结 backbone，以 self-consistency 学习 $\omega(c,s,t)$）
- 后续修正：CFG rescaling / dynamic thresholding（解决 oversaturation）、guidance distillation（消除 2× 推理开销）

## 在 text-guided editing 中的作用

- 几乎所有编辑方法默认开启 CFG；引导强度、对正/负 prompt 的差异化加权本身就是一个可调研的编辑杠杆
- Negative prompt 本质上也利用 CFG 框架：用不想要的条件做 $\varepsilon_u$ 的替代

## 出处与引用

- [[wiki/sources/hoClassifierFreeDiffusionGuidance2022]]（CFG 原文）
- [[wiki/sources/kynkaanniemiApplyingGuidanceLimited2024]]（time/noise-dependent guidance interval）
- [[wiki/sources/chidambaramWhatDoesGuidance2024]]（guided dynamics、极端区域集中与 score-error degradation）
- [[wiki/sources/chungCFGMANIFOLDCONSTRAINEDCLASSIFIER]]（CFG++ sampler update）
- [[wiki/sources/galashovLearnGuideYour2025]]（condition- / step-dependent learned guidance）
- [[wiki/sources/dhariwalDiffusionModelsBeat2021]]（classifier guidance 前身）
- 关联人物：[[wiki/entities/jonathan-ho]]
