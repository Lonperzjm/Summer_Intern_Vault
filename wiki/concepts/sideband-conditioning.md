---
type: concept
title: Sideband Conditioning（旁路条件注入 / frozen backbone + trainable sideband）
aliases: [sideband conditioning, sideband injection, frozen backbone sideband, 旁路条件注入, 旁路控制]
tags: [parameter-efficient-finetuning, conditioning, controlnet, lora, adapter, diffusion]
status: stable
created: 2026-05-28
updated: 2026-05-29
sources: ["[[wiki/sources/zhangAddingConditionalControl2023]]"]
---

# Sideband Conditioning（旁路条件注入）

## 一句话定义

**冻结预训练主干**（diffusion U-Net、文本编码器等），在主干之**侧**新增一个 trainable 模块（"sideband"），把新条件 / 新能力作为**加性残差**注入主干的中间表征或输出——主干参数不动、新模块用极小数据训。是 LoRA / Adapter / [[wiki/methods/controlnet|ControlNet]] / T2I-Adapter / IP-Adapter 等 PEFT (parameter-efficient fine-tuning) 方法的共同抽象。

## 抽象配方

$$
\varepsilon_\theta^{\text{tuned}}(z_t, t, c_t, c_{\text{new}}) = \varepsilon_\theta^{\text{frozen}}(z_t, t, c_t) + \mathcal S_\phi(z_t, t, c_t, c_{\text{new}})
$$

- $\varepsilon_\theta^{\text{frozen}}$：预训练大模型（如 [[wiki/entities/stable-diffusion|SD]]），**参数冻结**；
- $\mathcal S_\phi$：sideband 模块，**参数 $\phi$ 可训**；
- $c_{\text{new}}$：新引入的条件（空间图 / 图像 prompt / 风格 token / 任务标签 …）；
- **初始化约定**：$\mathcal S_\phi^{(0)}\equiv 0$（恒等元）——通过 [[wiki/concepts/zero-convolution|zero convolution]] / LoRA $A=0$ / adapter bottleneck 中的零初始化等手段实现，保证训练第 0 步整网输出 = 主干输出，不破坏已训练知识。

## 三种代表落点

| 方法 | $\mathcal S_\phi$ 接入位置 | $\mathcal S_\phi$ 结构 | 初始化恒等机制 | 主要服务的条件类型 |
|---|---|---|---|---|
| **LoRA** (Hu et al. 2022) | 主干内部线性层旁路（attn 的 Q/K/V/O、FFN 等） | 低秩 $\Delta W=BA$，$r\sim 4{-}64$ | $A=0$ 高斯，$B$ 高斯 | 风格 / 个性化 / 任务标签 |
| **Adapter** (Houlsby et al. 2019) | 主干内部 transformer block 后 | down-proj → nonlin → up-proj 瓶颈 | 末端 up-proj 零初始化 | NLP 任务迁移；视觉 PEFT |
| **[[wiki/methods/controlnet\|ControlNet]]** | 主干**外部** sideband 副本 → 加到 12 条 skip + middle | 克隆 SD U-Net 的 encoder + middle | [[wiki/concepts/zero-convolution\|zero conv]] | 空间对齐强条件（edge/depth/pose/...） |
| T2I-Adapter（待 ingest） | 同 ControlNet（加到 SD encoder） | 更小的 conv stem（不复制 SD encoder） | 类似 zero init | 空间对齐条件（轻量版） |
| IP-Adapter（待 ingest） | 主干 cross-attention 旁路 | 把图像 prompt 经 image encoder 后注入额外 cross-attn | 类似零初始化 | 图像 prompt |

> **共同范式抽象**：**frozen backbone + trainable sideband + 初始化恒等**。差别只在 (i) sideband 接入主干的位置；(ii) sideband 的内部结构与容量；(iii) 服务的条件 / 任务类型。

## 与其他概念的关系

- **vs 全模型 fine-tune**：sideband 不动主干 → 几乎不可能灾难性遗忘；50k–200k 数据即可稳；多个 sideband 可同时挂载（multi-condition）。
- **vs [[wiki/concepts/cross-attention|cross-attention 注入]]**：cross-attention 是 SD 主干**内部**的条件接口（token 类条件 K/V 化）；sideband 是主干**外部**接口（任意结构条件 + 任意位置注入）。两者**正交且通常共存**——SD 文本走原 cross-attn，pose/depth 走 ControlNet sideband。
- **vs [[wiki/concepts/classifier-free-guidance|CFG]]**：CFG 是采样期**逐 $t$** 对 conditional / unconditional ε 的线性外推；sideband 是网络结构层面的**全 $t$** 加性残差。两者完全正交。
- **vs [[wiki/concepts/perceptual-compression|perceptual compression]] 一档**：LDM 在 diffusion 之**前**加一次性预处理层；sideband 在 diffusion 之**侧**加一次性挂件——两条都印证 [[wiki/overview]] 推论 1 "范式不变、组件可加" 的思想（在 diffusion 算法本身之外新增组件，不动核心数学）。

## 在 text-guided editing 中的作用

**这是 thesis 在资源约束下的核心可行抽象**：

- 在 SD/SD3/FLUX 的 backbone 整段冻结的假设下（学界事实成立），text-guided editing 几乎所有方法都可重新归到 "sideband + 注入接口" 这一抽象之下：
  - **Inversion / noising-based**（✅ [[wiki/methods/sdedit|SDEdit]] / DDIM-inversion / Null-text inversion）：sideband = "在初始 noisy 状态上设置/优化的扰动"。**注意 SDEdit 是极端朴素的一支——连优化都没有**，直接把 guide 加噪到 [[wiki/concepts/noising-strength|$t_0$]] 当起点。所以"sideband"在此退化为"用 guide 替换初始噪声"，是抽象的**零成本下界**（对比 Null-text 要优化空文本 embedding 几百步）
  - **Attention-injection**（Prompt-to-Prompt / Plug-and-Play）：sideband = "在 cross/self-attention map 上加性 / 替换的修改"
  - **Control-injection**（[[wiki/methods/controlnet|ControlNet]] / T2I-Adapter）：sideband = "在 skip connection 上加性的空间条件残差"
- 差别只在 sideband 接入的位置（初始噪声 / attention map / skip connection）与训练成本（**SDEdit 零成本加噪 → DDIM-inversion 确定性反演 → Null-text 优化几百步 → ControlNet 50k–200k 训练**）。
- 对 [[wiki/overview]] 主要派系的影响：新增第 5 类 "Control/sideband-injection"，并把"frozen backbone + trainable sideband"作为四类编辑方法的**统一上层抽象**——这条统一化是 thesis 方法论范围的核心声明。

## 出处与引用

- 主要出处：[[wiki/sources/zhangAddingConditionalControl2023]]（ControlNet，sideband + zero-conv 的代表）
- 思想源流：LoRA (Hu et al. 2022，待 ingest)、Adapter (Houlsby et al. 2019，待 ingest)、Side-Tuning (Zhang et al. 2020，待 ingest)
- 同期 / 并行（待 ingest）：T2I-Adapter (Mou et al. 2023)、IP-Adapter (Ye et al. 2023)、GLIGEN (Li et al. 2023)
