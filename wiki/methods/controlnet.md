---
type: method
title: ControlNet
aliases: [ControlNet, control-net]
tags: [diffusion, control-injection, sideband, latent-diffusion, editing-related, foundational]
status: stable
created: 2026-05-28
updated: 2026-05-28
sources: ["[[wiki/sources/zhangAddingConditionalControl2023]]"]
family: other
---

# ControlNet

> family 标作 `other`：ControlNet 是 [[wiki/overview]] 主要派系**第 5 类 "Control/sideband-injection"** 的奠基与代表方法（不是 attention-injection、不是 inversion-based），但 vault 现行 family 字段尚未细化到这一档；保留 `other` 以保持兼容。

## 一句话总结

给 [[wiki/entities/stable-diffusion|Stable Diffusion]] 的 U-Net **encoder + middle** 段做 trainable copy，副本接收 noisy latent ⊕ 空间条件图，输出经 **[[wiki/concepts/zero-convolution|zero convolution]]** 加回原 U-Net 的 12 条 skip + 1 个 middle——**冻结主干、外挂 sideband、训练目标不变**，把强空间条件（Canny / depth / pose / segmentation / scribble / …）注入预训练 T2I 模型。

**抽象 block（原文 Fig 2）**：

![[zhangAddingConditionalControl2023-1779969426205.webp]]

左 (a) 原 frozen block $y=\text{block}(x)$；右 (b) ControlNet 升级：frozen block 不动，旁挂 trainable copy + 两端 zero conv，得 $y_c=y+\text{ZeroConv}(\text{TrainableCopy}(x+\text{ZeroConv}(c)))$。初始化 $y_c=y$（恒等，不破坏 frozen 输出），训练后控制 residual 生效。SD 上的 ControlNet 即把这个抽象 block 套在 U-Net encoder + middle 的各分辨率层级上——详见下方 Pipeline 与原文 Fig 3 全景图（在 [[wiki/sources/zhangAddingConditionalControl2023]]）。

## 核心机制

| 组件 | 内容 |
|---|---|
| 主干 | [[wiki/entities/stable-diffusion]] U-Net（**整段 frozen**：encoder + middle + decoder） |
| Trainable copy | 复制 SD U-Net 的 **encoder + middle** 部分（12 + 1 个 blocks），**decoder 不复制** |
| 副本输入 | 条件图 $c_f$ 经小 conv stem 编码后 + noisy latent $z_t$ |
| 注入方式 | 副本各 block 输出经 **[[wiki/concepts/zero-convolution\|zero-init $1\times 1$ conv]]** 投影后**加性**注入原 U-Net 的对应 12 条 skip 与 middle block |
| 训练目标 | $\mathcal L=\mathbb E\|\varepsilon-\varepsilon_\theta(z_t,t,c_t,c_f)\|^2$（与 [[wiki/methods/ddpm\|DDPM]]/[[wiki/methods/ldm\|LDM]] ε-pred MSE 同构） |
| 推断时 | SD 与 ControlNet 联合前向；采样器（DDIM/Euler/...）与 SD 完全一致；[[wiki/concepts/classifier-free-guidance\|CFG]] 仍可走文本路径 |

## Pipeline

### Training

```
# 准备：从 Stable Diffusion v1.5 加载预训练权重
freeze(SD.U_Net)                          # SD 整段冻结
copy = clone(SD.U_Net.encoder + SD.U_Net.middle)   # trainable copy
for each block in copy: add zero_conv after block
stem = small_conv_stack()                 # 4-layer conv stem 把 c_f (512×512) 打到 latent grid (64×64)

repeat:
    (x, c_t, c_f) ~ data                  # x: 目标图; c_t: 文本; c_f: 条件图（Canny/depth/pose/...）
    z = SD.E(x); t ~ U{1,...,T}; ε ~ N(0, I)
    z_t = sqrt(ᾱ_t)·z + sqrt(1-ᾱ_t)·ε

    # 文本编码（走 SD 原有 cross-attention，不动）
    c_t_tokens = SD.text_encoder(c_t)

    # 副本前向（sideband）
    h = stem(c_f) + z_t                   # 第一层加和
    side_outputs = copy.forward(h, t, c_t_tokens)   # 12 encoder + 1 middle 的中间输出
    side_residuals = [zero_conv_i(s) for i, s in enumerate(side_outputs)]

    # SD 主干前向 + 注入
    ε̂ = SD.U_Net.forward_with_sideband(z_t, t, c_t_tokens, side_residuals)

    L = ‖ε - ε̂‖²
    update {copy.params, zero_conv.params, stem.params}    # 只更新 sideband
```

### Sampling（与 SD 完全相同）

```
z_T ~ N(0, I)
c_t_tokens = SD.text_encoder(c_t)
for t = T, T-Δ, ..., 1:
    [optional CFG over c_t] ε̂ = SD.U_Net.forward_with_sideband(z_t, t, c_t_tokens, sideband_from(c_f))
    z_{t-Δ} = DDIM-update(z_t, ε̂, t)
return SD.D(z_0)
```

### 实现要点

- **副本只复制 encoder + middle**：decoder 不复制；这是参数量减半且只学"如何向 skip 注入残差"的关键。
- **Zero convolution**：$1{\times}1$ conv，weight = 0, bias = 0 初始化（但 weight 是 trainable）；详见 [[wiki/concepts/zero-convolution]]。
- **Prompt dropout**：训练时 50% 概率把 $c_t$ 替换为空 prompt，让 ControlNet 学到 unconditional 控制能力（配合 SD 的 CFG）。
- **数据规模**：50k–1M 即可。原文报告 50k Canny 数据已足以稳。
- **多条件叠加**：多个 ControlNet 可同时挂载、加性 residual 直接叠加（pose + depth 等组合控制）。

## 适用场景与限制

**适用**：
- 给预训练 T2I 模型加**空间对齐**条件控制（edge / depth / pose / segmentation / scribble / normal / line drawing / …）
- 学界资源约束下的可行研究路径——单 3090/A100、50k 量级数据
- 与 [[wiki/concepts/cross-attention|cross-attention]] 文本注入**正交共存**

**限制**：
- 仍要训副本（虽然只副本一半 U-Net 也有数亿参数）—— 比 LoRA/T2I-Adapter 重
- "Sudden convergence phenomenon"（训练相位转换）尚无理论解释——训练稳定性诊断仍靠经验
- 条件图与目标图像的"对齐先验"假设——非完美对齐场景（如跨体型 pose 移植）的几何敏感度未充分研究
- 在 SD3 / FLUX (RF + DiT backbone) 上的可迁移性——skip 结构变化，sideband 注入位置需重新设计

## Failure modes

- 副本太浅（如只复制 encoder 前几层）→ 复杂空间条件学不进去（原文 §4.2 ablation）
- 把 zero-conv 替换成普通高斯初始化 → 训练崩溃
- multi-condition 强约束矛盾时（如 pose 与 depth 冲突）—— 论文未做详细 ablation
- 极端域偏移条件（如手绘草图 vs 真实 Canny）→ 副本可能要重训

## 关联

- 出处：[[wiki/sources/zhangAddingConditionalControl2023]]
- 概念基础：[[wiki/concepts/zero-convolution]]、[[wiki/concepts/sideband-conditioning]]
- 主干模型：[[wiki/entities/stable-diffusion]]；方法基础 [[wiki/methods/ldm|LDM]]
- 训练目标：[[wiki/concepts/epsilon-parameterization|ε-prediction]]（不动）；[[wiki/methods/ddpm|DDPM]] / [[wiki/methods/ddim|DDIM]] 采样器（不动）
- 正交接口：[[wiki/concepts/cross-attention]]（文本 token 注入）与 ControlNet（空间对齐 sideband 注入）通常共存
- 同期 / 并行（待 ingest）：T2I-Adapter (Mou et al. 2023)、IP-Adapter (Ye et al. 2023)、GLIGEN (Li et al. 2023)
- 下游 / 后续（待 ingest）：FLUX-ControlNet、SD3-ControlNet（RF backbone 上的可迁移性问题）
- 人物：[[wiki/entities/lvmin-zhang]]、[[wiki/entities/anyi-rao]]、[[wiki/entities/maneesh-agrawala]]；机构 [[wiki/entities/stanford]]
