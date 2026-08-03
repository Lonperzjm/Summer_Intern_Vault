---
type: source
title: "Scalable Diffusion Models with Transformers (DiT)"
aliases: [DiT, "Peebles & Xie 2023", peeblesScalableDiffusionModels2023]
tags: [diffusion, transformer, architecture, scaling, class-conditional]
status: active
created: 2026-07-24
updated: 2026-07-24
raw: "[[raw/literature-notes/peeblesScalableDiffusionModels2023]]"
authors: [William Peebles, Saining Xie]
venue: ICCV
year: 2023
---

# Scalable Diffusion Models with Transformers (DiT)

## 一句话

用标准 Vision Transformer 替代 U-Net 作为扩散模型的噪声预测骨干网络，证明 Transformer 在图像生成中同样服从 scaling law——更多 Gflops 可以换取更低 FID。

## 背景与动机

扩散模型的骨干网络长期由 U-Net 主导（[[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]]、[[wiki/methods/ldm|LDM]]、ADM）。而 ViT 在判别式任务中已证明了良好的 scaling 特性。DiT 的核心问题：**把 U-Net 换成 Transformer 能否保持（甚至超越）生成质量，同时继承 Transformer 的可扩展性？**

## 方法

### 整体管线

DiT 在 [[wiki/methods/ldm|LDM]] 框架下工作：

```
输入图像 x ∈ ℝ^{256×256×3}
    → 预训练 VAE encoder E(·)  →  latent z ∈ ℝ^{32×32×4}
    → Patchify（patch_size p）  →  token 序列 T ∈ ℝ^{(32/p)²×d}
    → N 层 DiT Block（条件：timestep t、class label c）
    → Linear Decoder → 预测噪声 ε̂ (或 v-prediction)
```

使用 Stable Diffusion 同款 VAE（KL-reg, f=8），空间压缩 8× 后在 latent space 做扩散。

### Patchify 层

- 将 32×32×4 的 latent 切为 (32/p)² 个 patch，每个 patch 线性投影到 d 维
- p=2 → 256 tokens；p=4 → 64 tokens；p=8 → 16 tokens
- 加上标准的 **frequency-based positional embedding**（sin/cos）
- 更小的 p 意味着更多 tokens、更高 Gflops，但信息保留更完整

### DiT Block 设计：四种条件注入方式

论文系统对比了 4 种将 (t, c) 条件信息注入 Transformer block 的方式：

| 变体 | 机制 | 效果 |
|------|------|------|
| **In-context** | 将 t 和 c 的 embedding 作为两个额外 token 拼接到序列中，参与 self-attention | 最简单；性能最差 |
| **Cross-attention** | 将 (t, c) embedding 拼接为 KV 序列，在额外 cross-attention 层中注入 | 引入 ~15% 额外 Gflops；仍不及 adaLN |
| **adaLN (Adaptive Layer Norm)** | 用 (t, c) 回归 LayerNorm 的 γ, β 参数（类似 [[wiki/concepts/conditional-diffusion|条件扩散]] 中的 FiLM） | 几乎无额外参数；显著优于前两者 |
| **adaLN-Zero** | 在 adaLN 基础上额外回归一组 scale α（初始化为零），作用于残差分支输出 | **性能最优**；训练起步时每个 block 等价于 identity → 易优化 |

最终选择：**adaLN-Zero**。条件嵌入流程：
```
t_emb = MLP(sinusoidal_embed(t))      # timestep
c_emb = embedding_table(c)            # class label
cond  = t_emb + c_emb                 # 相加后共享
(γ₁, β₁, α₁, γ₂, β₂, α₂) = linear(SiLU(cond))   # 回归 6 个 scale/shift
```

### DiT Block 前向传播（adaLN-Zero）

```
h  = x + α₁ · Attention(adaLN(x; γ₁, β₁))
out = h + α₂ · FFN(adaLN(h; γ₂, β₂))
```

其中 adaLN(x; γ, β) = γ · LayerNorm(x) + β。α 初始化为 0 保证训练初期残差分支输出为零，整个 block 为恒等映射。

### 输出头

最终 DiT block 输出经过一个 **adaLN → Linear** 解码器，线性层输出维度 = p² × 2C（C=4 为 latent channels），对应噪声和协方差的预测：
- 前半输出：ε̂ 或 v 预测
- 后半输出：对角协方差 Σ（类似 IDDPM 的参数化）

## 模型配置

| 模型 | Layers N | Hidden dim d | Heads | Params | Gflops (p=2) |
|------|----------|-------------|-------|--------|--------------|
| DiT-S/2 | 12 | 384 | 6 | 33M | 6 |
| DiT-B/2 | 12 | 768 | 12 | 130M | 23 |
| DiT-L/2 | 24 | 1024 | 16 | 458M | 80 |
| DiT-XL/2 | 28 | 1152 | 16 | 675M | 119 |

"/2" 表示 patch_size=2（最密 tokenization）。

## 核心发现

### Scaling Law

- **Gflops 越高 → FID 越低**，关系近似 log-linear
- 这一趋势跨模型大小（S→XL）和 patch size（/8→/2）一致
- Gflops 是比参数量更好的性能预测指标（因为 patch size 影响 token 数而非参数量）
- 训练更久也无法弥补模型容量不足——大模型 400K steps 时已超过小模型 7M steps

### 定量结果（ImageNet 256×256，class-conditional，cfg=1.5）

| 模型 | FID-50K↓ | sFID↓ | IS↑ |
|------|----------|-------|-----|
| ADM (U-Net) | 10.94 | 6.02 | 100.98 |
| ADM-U | 7.49 | 5.13 | 127.49 |
| LDM-4 | 10.56 | — | 103.49 |
| **DiT-XL/2** | **2.27** | **4.60** | **278.24** |

ImageNet 512×512：DiT-XL/2 达到 FID **3.04**，同样 SOTA。

### Classifier-Free Guidance

DiT 使用 [[wiki/concepts/classifier-free-guidance|classifier-free guidance]]（cfg scale 1.5），训练时 10% 概率 drop class label → 学无条件分支。这是达到 2.27 FID 的关键——无 cfg 时 FID 约 9.62。

## 关系

- **基于** [[wiki/methods/ldm|LDM]] 的 latent space 设计（同 VAE、同 latent 分辨率）
- **替代** U-Net 骨干（DDPM/ADM 系列用 U-Net + attention）
- **条件机制** 类似 [[wiki/concepts/conditional-diffusion|FiLM / Adaptive Normalization]]，但加入 zero-init 技巧
- **采样** 使用 [[wiki/concepts/classifier-free-guidance|CFG]] 提升质量
- **影响了** 后续 Transformer-based 扩散模型：Stable Diffusion 3（MM-DiT）、FLUX、Sora 均采用 DiT 骨干或其变体
- **Scaling 哲学** 延续 ViT / GPT 系列：标准架构 + 规模 → 涌现质量

## 对我的 thesis 的启示

DiT 证明 Transformer 骨干在扩散模型中完全可行且更可扩展，这对 text-guided image editing 有两层影响：
1. **架构基础**：当前 SOTA editing 模型（FLUX-Kontext 等）已采用 DiT/MM-DiT 骨干，理解 DiT 是理解这些方法的前提
2. **条件注入**：adaLN-Zero 的 "zero-init → identity at start" 设计思想在 [[wiki/methods/controlnet|ControlNet]] 的 zero-convolution 中有呼应；在 energy-guidance 研究方向中，guidance signal 的注入方式可参考此处的对比实验

暂不需要更新 [[wiki/overview]] 的 working thesis——DiT 属于 backbone-level 背景知识，非直接的 editing 方法。

## 我的 takeaways

1. **adaLN-Zero 是最优条件注入方式**：zero-init 让训练初期 block 为 identity，优化 landscape 更平滑。这一 insight 在后续工作中被广泛复用。
2. **Gflops 是扩散模型 scaling 的正确度量轴**：参数量不够——因为 token 数（由 patch size 决定）大幅影响计算量但不影响参数量。
3. **Latent diffusion + Transformer = 完整故事**：VAE 负责像素空间压缩（一次训练、多任务复用），Transformer 负责语义空间中的扩散过程。两者解耦是 scaling 的前提。
4. **CFG 对 Transformer 骨干同样有效**：10% label drop + inference-time cfg=1.5 可将 FID 从 9.62 降至 2.27。
5. **DiT 为后续 MM-DiT、Sora 等工作奠基**：证明了 "用 Transformer 做扩散" 不仅可行，而且在足够规模下优于 U-Net。
