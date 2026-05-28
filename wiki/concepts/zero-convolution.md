---
type: concept
title: Zero Convolution（零初始化卷积）
aliases: [zero convolution, zero conv, zero-init conv, 零初始化卷积, "ZeroConv"]
tags: [parameter-efficient-finetuning, stabilization, sideband, controlnet]
status: stable
created: 2026-05-28
updated: 2026-05-28
sources: ["[[wiki/sources/zhangAddingConditionalControl2023]]"]
---

# Zero Convolution（零初始化卷积）

## 一句话定义

一个 $1\times 1$ 卷积层，**weight = 0、bias = 0** 初始化（但参数是 trainable）；用作 sideband 模块到主干的"残差注入端口"——**初始化时该模块整体表现为恒等映射**（不扰动主干输出），训练时梯度仍能流通（因输入非零），逐步学到非零残差。是 [[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 稳定 fine-tune 大预训练模型的核心机制。

**视觉直观（[[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 原文 Fig 2）**：

![[zhangAddingConditionalControl2023-1779969426205.webp]]

右 (b) 中两处 "zero convolution" 是本概念的物理位置——一处在条件 $c$ 进入 trainable copy **之前**（把 $c$ 安全地"封口"成 0 输入到副本第一层），一处在副本输出回到 frozen block **之后**（把副本输出"封口"成 0 残差加回主干）。初始化时这两处都输出 0，所以 $y_c=y+0=y$；训练后两处都学到非零权重。

## 数学/技术细节

设 zero-conv 为 $\mathcal Z(x;W,b)=W\ast x+b$，$x\in\mathbb R^{B\times C\times H\times W}$，$W\in\mathbb R^{C'\times C\times 1\times 1}$。

**初始化**：$W=0,\,b=0$。

**前向第 0 步**：对任意输入 $x$，$\mathcal Z(x)=0\cdot x+0=0$。所以挂载它的 sideband 残差为 0，整网输出 = 主干（frozen SD）输出，损失 = 主干在该 batch 上的损失。

**梯度第 0 步**：
$$
\frac{\partial \mathcal L}{\partial W} = \frac{\partial \mathcal L}{\partial \mathcal Z}\cdot x^\top,\qquad \frac{\partial \mathcal L}{\partial b} = \frac{\partial \mathcal L}{\partial \mathcal Z}\cdot \mathbf 1.
$$

只要 (i) $\partial\mathcal L/\partial\mathcal Z\ne 0$（这由主干 + skip 结构决定，一般成立）；(ii) **输入 $x\ne 0$**（由 sideband 副本前一层的非零激活决定），梯度即非零。第一步更新 $W\leftarrow W-\eta\,\partial\mathcal L/\partial W\ne 0$，自此打破对称性，后续训练正常进行。

**第 1 步起的"额外梯度通道"**：一旦 $W\ne 0$，
$$
\frac{\partial\mathcal Z}{\partial x}=W\ne 0,
$$
所以 sideband 副本的上游参数也能收到来自主干 loss 的梯度——副本得以正常学习。

> 关键洞察：zero-conv 是 **"初始化输出 = 0、但初始化梯度 $\ne 0$"** 的设计。这与一般"零初始化全网络"的训练崩溃截然不同（后者全梯度也为 0）——zero-conv 只把**输出端**置零、保持**输入端非零**。

## 与其他概念的关系

### vs LoRA 的零初始化矩阵 $A=0$

LoRA 把权重增量写成 $\Delta W=BA$，初始化 $A=0$（高斯）、$B=$ 高斯（也常见反过来）。同样满足"初始化 $\Delta W=0$ → 主干输出不被扰动"，但梯度结构不同：

| | Zero Conv | LoRA |
|---|---|---|
| 参数化形式 | $W$ 单层卷积 | $\Delta W = BA$ 两矩阵积 |
| 零初始化对象 | $W=0,\,b=0$ | $A=0$（保留 $B$ 高斯） |
| 第 0 步输出 | 0 | 0（$BA=0$） |
| 第 0 步梯度 | $\partial\mathcal L/\partial W\propto x$ 即可 | $\partial\mathcal L/\partial B=0$（因 $\partial(BA)/\partial B=A=0$），但 $\partial\mathcal L/\partial A\propto B$ 可学 |
| 注入位置 | 主干外部 sideband 与 skip 加性融合 | 主干内部线性层旁路 $W+\Delta W$ |

两者**思想同源**：让新模块在初始化时为恒等元，避免破坏预训练知识；技术细节按是否 sideband / 是否低秩选择不同形式。

### vs 普通高斯/Xavier 初始化的 $1\times1$ conv

ControlNet 原文 §4.2 ablation：把 zero-conv 替换为标准高斯初始化的 $1\times 1$ conv，**训练崩溃**（早期 sideband 残差是随机噪声，扰动 SD 已收敛的输出，loss 不降反升）。

### vs Identity-init / Skip-residual

ResNet 风格 "$y=x+f(x)$" 也有"初始化 $f=0$ 即为恒等"的味道，但 ResNet 的 $f$ 是从零训练，不存在"保护已训练主干"的诉求。Zero-conv 是这条思想在 **预训练模型 + 新加 sideband** 场景下的延伸：保护的是已训好的主干，不是一个空的随机初始化。

## 在 text-guided editing 中的作用

- **直接基础**：[[wiki/methods/controlnet|ControlNet]] 的稳定 fine-tune 关键——50k 数据规模能稳，主要靠 zero-conv 防止早期破坏；
- **可迁移到其他 sideband 注入**：T2I-Adapter（待 ingest）也用类似初始化思想（虽然实现略有不同）；任何"在预训练 T2I 模型外挂控制分支"的方法都该考虑 zero-init 或同类思想；
- **对 thesis 的含义**：是 [[wiki/concepts/sideband-conditioning|frozen backbone + sideband 范式]]的稳定性"硬件"——理论上不止 ControlNet，所有该范式下的方法都需要某种 "初始化恒等" 的机制（zero-conv / LoRA $A=0$ / adapter 内部 bottleneck 等都是其变体）。

## 出处与引用

- 主要出处：[[wiki/sources/zhangAddingConditionalControl2023]] §3.1 / §4.2
- 思想源流：LoRA (Hu et al. 2022，仍待 ingest)、Adapter (Houlsby et al. 2019，待 ingest) 中的"初始化恒等"思路；ResNet 残差链中的 identity 元素
