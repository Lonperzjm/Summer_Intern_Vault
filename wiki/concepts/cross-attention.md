---
type: concept
title: Cross-Attention（跨注意力，作为条件注入接口）
aliases: [cross-attention, cross-attn, 跨注意力]
tags: [attention, conditioning, diffusion, transformer]
status: stable
created: 2026-05-27
updated: 2026-05-28
sources: ["[[wiki/sources/rombachHighResolutionImageSynthesis2022]]"]
---

# Cross-Attention（条件注入接口）

## 一句话定义

把"标准 self-attention"中 K/V 的来源换成**外部条件序列** $\tau_\theta(y)$（而 Q 仍来自主流 feature）：让主流的每个空间位置**自适应地从条件 token 序列中检索相关信息**——是把 text / layout / class 等 token 类条件注入 [[wiki/methods/ddpm|diffusion]] U-Net 的事实标准。

## 数学/技术细节

[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] §3.3 给出：

$$
\mathrm{Attention}(Q,K,V) = \mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt d}\right)V,
$$
$$
Q = W_Q^{(i)}\!\cdot\!\phi_i(z_t),\quad
K = W_K^{(i)}\!\cdot\!\tau_\theta(y),\quad
V = W_V^{(i)}\!\cdot\!\tau_\theta(y).
$$

- $\phi_i(z_t)\in\mathbb R^{N\times d_\epsilon^i}$：U-Net 第 $i$ 层的空间 feature，flatten 后 $N=h_i w_i$ 个 token，每个 token 即一个空间位置。
- $\tau_\theta(y)\in\mathbb R^{M\times d_\tau}$：条件编码器输出（文本走 transformer，layout token 化等），$M$ 是条件 token 数（文本即 token 数，~77 for CLIP）。
- $W_{Q,K,V}^{(i)}$：每个注入层独立的线性投影；通常嵌在 U-Net 中间块的 transformer block 内（self-attn → **cross-attn** → FFN 的标准 transformer 模式）。

### Attention map：$N\times M$，编辑算法的"语义到空间"显式接口

记 $A^{(i)} = \mathrm{softmax}(QK^\top/\sqrt d)\in\mathbb R^{N\times M}$。

- $A^{(i)}_{n,m}$ = 第 $n$ 个空间位置对第 $m$ 个条件 token 的注意力权重；
- 给定一个条件 token（如文本中的"cat"），$A^{(i)}_{:,m}$ 即该 token 在该层、该时间步上的**空间热图**——很多 attention-injection 编辑方法（Prompt2Prompt、Plug-and-Play 等）直接读 / 写 / 重映射这张图来实现"局部替换"、"风格保持"等。

## 与其他概念的关系

- **vs Self-Attention**：Q/K/V 都来自同一序列；Cross-Attention 把 K/V 替换为外部条件。两者通常**交替使用**——LDM 的 U-Net 中每个 transformer block 是 (self-attn) → (cross-attn) → (FFN)。
- **vs Conditional Drift（条件 LDM）**：cross-attention 是**实现** conditional drift $\varepsilon_\theta(z_t, t, \tau_\theta(y))$ 的具体机制；理论上也可以走 concat / AdaGN / FiLM，但 cross-attention 在 token 数可变、模态切换、attention map 可读性上明显占优。
- **vs Concat 注入（空间对齐条件）**：[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] 中把**空间对齐**条件（semantic map / 低清图 / mask）直接 concat 到 noisy latent 沿通道维，**不走** cross-attention——因为这类条件本身已是 spatial grid，无需"token-空间"检索；cross-attention 主要负责"non-spatial token 序列 ↔ 空间"的对齐。
- **vs Sideband 注入（[[wiki/methods/controlnet|ControlNet]] 系）**：cross-attention 在 U-Net **内部** transformer block 里把 token 类条件 K/V 化注入；ControlNet 在 U-Net **外部** sideband 副本中把空间对齐条件加性 residual 化注入到 12 条 skip + middle。两者**正交且共存**——SD 上文本走原 cross-attn、pose/depth 走 ControlNet sideband。统一抽象见 [[wiki/concepts/sideband-conditioning]]。
- **vs [[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance]]**：**正交**。Cross-attention 给出 conditional drift $\varepsilon_\theta(\cdot, c)$；CFG 在采样期对 conditional / unconditional drift 做线性外推 $\varepsilon_\theta(\cdot,c) + s(\varepsilon_\theta(\cdot,c)-\varepsilon_\theta(\cdot,\varnothing))$。SD 实践把两者联用——CFG 仍以 cross-attention 给出的 conditional ε 为底料。
- **vs [[wiki/concepts/classifier-guidance|Classifier Guidance]]**：classifier guidance 走贝叶斯拆分 $\nabla\log p(y\mid x_t)$（需要训一个 noisy classifier）；cross-attention 把 $y$ 当作网络输入，**不**做贝叶斯拆分——这是用户在 ε-pred 高亮处的批注「score 的贝叶斯公式之外的另一种方法」精确指向的差异。

## 在 text-guided editing 中的作用

- **核心枢纽**：cross-attention map 是 SD 类模型中**唯一可被显式读写的"语义—空间"接口**，几乎所有 attention-injection 编辑方法都围绕它展开：
  - **Prompt-to-Prompt**：跨时间步替换 / 重权 attention map
  - **Null-text inversion**：优化空文本 embedding 使 cross-attn 重建对齐
  - **Plug-and-Play (PnP-Diffusion)**：注入 self-attn + cross-attn 的中间 feature
  - **Attend-and-Excite**：检测"漏注意力" token 并梯度调高
  - **MasaCtrl**：mutual self-/cross-attn 控制
  - **StyleAligned**：共享 cross-attn 的 K/V 实现风格一致
- 编辑论文常引"cross-attention 是 LDM 的语义瓶颈" —— 攻击它即可在不重训的前提下控制生成内容。

## 出处与引用

- 主要出处：[[wiki/sources/rombachHighResolutionImageSynthesis2022]] §3.3 eq. (1)
- 上游概念：Transformer 的 multi-head attention（Vaswani et al. 2017）
- 下游编辑：上述编辑论文族（多数仍待 ingest）
