---
type: synthesis
title: 领域总览 · Diffusion / Flow Matching for Text-Guided Image Editing
tags: [overview, thesis]
status: active
created: 2026-05-05
updated: 2026-06-24
sources: ["[[wiki/sources/hoDenoisingDiffusionProbabilistic2020]]", "[[wiki/sources/songDenoisingDiffusionImplicit2022]]", "[[wiki/sources/songScoreBasedGenerativeModeling2021]]", "[[wiki/sources/lipmanFlowMatchingGenerative2023]]", "[[wiki/sources/liuFlowStraightFast2022a]]", "[[wiki/sources/rombachHighResolutionImageSynthesis2022]]", "[[wiki/sources/zhangAddingConditionalControl2023]]", "[[wiki/sources/mengSDEditGuidedImage2022]]", "[[wiki/sources/zhouDenoisingDiffusionBridge2023]]", "[[wiki/sources/albergoStochasticInterpolants2023]]", "[[wiki/sources/zhaoEGSDEUnpairedImagetoImage2022]]", "[[wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b]]"]
---

# Overview

本页是整个 wiki 的"枢纽"，由 Claude Code 在 ingest 新源后持续更新。

## Working Thesis（v0.5，基于 DDPM + DDIM + Score SDE + Flow Matching + DDBM/Bridge）

> Diffusion 模型把图像生成建模为**全局 + 渐进**的去噪过程：每一步对整张图作用（全局），但分 T 步逐级精化（渐进）。其工程化的关键在于 [[wiki/concepts/epsilon-parameterization]]——把训练目标从"预测 reverse 均值"改写为"预测注入噪声 ε"，并配合极简 L2 损失 $L_\mathrm{simple}$。这一参数化与 [[wiki/concepts/score-matching]] 在尺度因子下等价，从而把 [[wiki/sources/hoDenoisingDiffusionProbabilistic2020|DDPM]] 与 [[wiki/methods/ncsn|NCSN 系（Song & Ermon 2019）]]统一在 score-based 框架下。
>
> [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE（Song et al. 2021）]] 把这个"统一"从类比抬升为**严格的连续时间理论**：前向加噪是一条 SDE，DDPM = VP-SDE、NCSN = VE-SDE 只是它的两种离散化（[[wiki/concepts/score-sde]]）；反向生成只依赖 score，且**同一个 score 网络**可被三种采样器复用——随机的反向 SDE、带校正的 [[wiki/concepts/predictor-corrector-sampling|predictor-corrector]]、确定性的 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]。这给下面的"可变性光谱"提供了迄今最干净的支撑：**训练目标固定，采样链却可以整段替换**。
>
> 推论（待后续源验证 / 演化）：
>
> 1. **真正的不变量是"范式"，不是某个"层"；组件落在可变性光谱上**
>
>    与其把技术栈切成"固化层 / 编辑层"，不如承认：唯一近乎不变的是**范式**——
>    > *迭代式生成 + 网络预测噪声 / 速度场 + 沿生成链注入条件*。
>
>    其余一切组件按"被改动的频率与代价"排在一条光谱上：
>
>    | 可变性 | 组件 | 说明 |
>    |---|---|---|
>    | 近乎不变 | 上述范式本身 | 从 DDPM 到 FLUX 都成立 |
>    | 可演化，但非主战场 | 训练目标（ε → v → 速度场）、backbone（U-Net → DiT）、**压缩层**（pixel → latent，见下） | 改它性价比低、要和生态对抗；但**确实在变** |
>    | 研究杠杆 | inversion 质量、guidance 形式、条件注入通道、介入时间步 | 论文创新点集中于此 |
>
>    **关于"压缩层"这一档（2026-05-27 [[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] ingest 后新增）**：与其它三档不同，压缩层是**正交于 diffusion 算法本身**的——LDM 没动训练目标、没动 backbone 类型、没动采样器，只在所有 diffusion 操作之前**外加一个一次性预处理层** $(\mathcal E,\mathcal D)$（[[wiki/concepts/perceptual-compression]]）。它给"组件维度"补了一档，但**不影响**"训练目标 / backbone / 采样器"那三档的可变性序列。对 thesis 的实际意义见推论 2 的扩展注。
>
>    **关键修正**：原来的"基础设施层 / 编辑层"两层划分有三个硬伤——(i) 把 U-Net 当作固化组件是错的，backbone 一直在被换（DiT、SD3、FLUX）；(ii) "底层一字不改"与 Flow Matching 换训练目标自相矛盾；(iii) 最致命：两层划不开——"条件注入"这个杠杆**本身就要触及 backbone 内部**（[[wiki/concepts/cross-attention|cross-attention]] 在 U-Net 里、[[wiki/methods/controlnet|ControlNet]] 是 U-Net 旁路副本——不动 backbone 参数但克隆 backbone 结构作 sideband，详见 [[wiki/concepts/sideband-conditioning]]）。所以 backbone 不是"与编辑无关的下层"，而是**研究杠杆施力的对象之一**。
>
>    **对 thesis 的含义**：不要把"重新设计训练目标"当成主战场（性价比低，要和整个生态对抗）；可行的研究杠杆是 inversion 质量、guidance 形式、条件注入通道、介入时间步——但要清醒：动"条件注入"往往意味着动 backbone 架构本身。
>
>    **支持光谱"中间一档"的证据**（下次 ingest 时继续验证）：
>    - ✅ **[[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]]（已 ingest 2026-05-24）给出最强样本**：FM **真的换了训练目标**（[[wiki/concepts/conditional-flow-matching|CFM]] 回归速度场而非 ε/score），却仍落在"迭代生成 + 预测速度场 + 沿链注入条件"的范式里；同架构消融下 FM-OT 在 NLL/FID/NFE 三项均超 [[wiki/methods/ddpm|DDPM]]。下游 Rectified Flow（FLUX、SD3）继承其训练目标，其上的编辑方法（RF-Inversion 等）依然在 ODE 链上注入条件——"训练目标可演化、范式不变"在此最清楚。
>    - v-prediction、EDM preconditioning、Min-SNR weighting 等是对 $L_\mathrm{simple}$ 的"调参级"修改，能做小幅 ablation 但远未触及"重新设计目标"的级别。
>    - 📐 **"训练目标"这一档的数学本质（[[wiki/comparisons/score-vs-velocity-field]]）**：从 score 到 velocity field 不只是"换个回归目标"——score $\nabla\log p_t$ 必是**保守场**，FM 的 $v_t$ 一般是**非保守场**（写不成 $\nabla(\cdot)$）。所以这一档的可演化性本质是**把生成动力学从"保守速度场子集"解放到"一般速度场"**；diffusion 是 FM 在"保守场 + 高斯路径"特例上的退化。这也解释了为何 [[wiki/methods/rectified-flow|RF]] 能拉直而 score 流不能（直线速度场非保守）。
>    - **✅ [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] 给出最干净的样本**：训练好一个 score 网络后，可在**不重训**的前提下换三种采样器（反向 SDE / [[wiki/concepts/predictor-corrector-sampling|PC]] / [[wiki/concepts/probability-flow-ode|probability-flow ODE]]）。这说明"采样链的形状"整段坐落在光谱的"研究杠杆 / 介入方式"一档——可演化、代价低、不碰训练目标。光谱因此可细化为：**训练目标（最贵）> backbone > 采样器 / guidance / 介入时间步（最便宜）**。
>    - **✅ [[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] 给出"压缩层"这一新档**：LDM 把扩散搬到 autoencoder 给出的 latent 空间，**训练目标、backbone 类型、采样器全都未动**——只是新引入了一个一次性预处理层 $(\mathcal E,\mathcal D)$。这是对推论 1 的**额外维度**支撑：组件可以"新增"而非只是"替换"，但 paradigm 仍不动。后果是：SD3 / FLUX = LDM 的压缩层 ⊕ FM/RF 的训练目标——两条**正交**演化线在工业实现处汇合。
>    - **✅ [[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 给出"sideband 注入"这一新档**：ControlNet 进一步把"组件可新增"推到极端——它**不修改 backbone 参数**（freeze SD），但**克隆了 backbone 内部结构**（SD encoder + middle 的 trainable copy）作为外挂；新条件通过 [[wiki/concepts/zero-convolution|zero conv]] 加性 residual 注入原 U-Net 的 skip。这条与 LDM "压缩层"对称——一个在 diffusion **之前**加，一个在 diffusion **之侧**加，两条都印证"paradigm 不动，组件可加"。统一上层抽象见 [[wiki/concepts/sideband-conditioning|frozen backbone + trainable sideband]]——这条**抽象本身**很可能成为 thesis 的方法论根线（详见推论 1 后半段）。
>
> 2. **全局-渐进权衡**：编辑保真度 vs 可控性 trade-off 很可能与"在 reverse 链哪一段（哪一时间步）介入"强相关——高 $t$ 改变结构，低 $t$ 改变细节。✅ **这条假设已被 [[wiki/sources/mengSDEditGuidedImage2022|SDEdit]]（2026-05-29 ingest）首次直接验证**（见下）。
>
>    **Score SDE 给了这个假设一个形式化抓手**：其条件反向 SDE 由贝叶斯拆分 $\nabla_x\log p_t(x\mid y)=\nabla_x\log p_t(x)+\nabla_x\log p_t(y\mid x)$ 得到，引导项 $\nabla\log p_t(y\mid x)$ 是**逐时间步 $t$** 显式加上去的（[[wiki/concepts/score-sde]] / [[wiki/concepts/classifier-free-guidance]]）。于是"在哪个 $t$ 注入、注入多强"从经验调参变成连续可写的量——正是 fidelity↔controllability 旋钮的数学形态。
>
>    **✅ 首个直接编辑实证（[[wiki/sources/mengSDEditGuidedImage2022|SDEdit]] 引入）**：此前推论 2 只有 Score SDE 的形式化抓手、无编辑论文的经验支撑。SDEdit 补上了——它把"从哪个 $t_0$ 开始 reverse"作为唯一旋钮（[[wiki/concepts/noising-strength|noising strength]]），Fig 3 实验**直接画出** $t_0$↑ ⇒ realism↑ / faithfulness↓ 的单调曲线，把"在哪个 $t$ 介入"从假设变成可测量。两个细节值得 thesis 注意：(i) SDEdit 的 $t_0$ **同时**编码"介入时间步"与"加噪强度"（两旋钮耦合成一个），与 [[wiki/concepts/classifier-free-guidance|CFG]] 的"逐 $t$ 引导强度 $w$"是**不同旋钮**——前者调起点、后者调每步方向放大，两者可组合、是否正交是 open question；(ii) SDEdit 是 $t_0$ **单次介入**（之后纯 reverse），与 [[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 的全 $t$ sideband 构成「单次 vs 全程介入」两端点。
>
>    **⚠️ 跨范式 caveat（[[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 引入）**：FM 的 $t$ 是噪声→数据的**插值系数**（$t{=}0$ 噪声、$t{=}1$ 数据），与 diffusion 的 $t$ **方向相反、语义不同**。讨论"在哪个时间步介入"时必须先统一坐标，否则"高 $t$ 改结构、低 $t$ 改细节"这类结论在 FM 上会整体翻转。
>
>    **⚠️ 正交维度（[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] 引入）**：现有 fidelity↔controllability 形式化（"在哪个 $t$ 注入 + 注入多强"）默认信号在 diffusion 前是**无损**的；但 LDM/SD 上**所有**操作都发生在 latent $z=\mathcal E(x)$ 上，[[wiki/concepts/perceptual-compression|autoencoder 重建误差]] $\mathcal D(\mathcal E(x))-x$ 是 **fidelity 的 hard upper bound**——独立于 $t$、独立于 guidance 强度。这给 thesis 多了一个**更上游的、正交的** fidelity 变量。后果之一：DDIM inversion 在 SD 上的失败模式可能与 autoencoder 失真**纠缠**，是 thesis 可立的一个干净诊断维度（"绕过 $\mathcal E,\mathcal D$ 的 pixel 空间对照"实验已记入 [[wiki/sources/rombachHighResolutionImageSynthesis2022]] open questions P0）。
>
>    **⚠️ 全 $t$ vs 逐 $t$ 注入的对照（[[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 引入）**：推论 2 把 fidelity↔controllability 旋钮形式化为"在哪个 $t$ 注入条件 + 多强"——这隐含**逐 $t$ 注入**的 [[wiki/concepts/classifier-free-guidance|CFG]] 风格。但 ControlNet 是**时不变的全 $t$ 加性 sideband**：它在所有 $t$ 都把同一份 residual 加到 skip 上，并没有"哪个 $t$ 注入"这个旋钮。两者经验上都能用、且 ControlNet 比 prompt-only 编辑更**结构保真**——thesis 因此要立的新假设：**"全 $t$ sideband"与"逐 $t$ guidance"在 fidelity↔controllability 旋钮上的差异**是否系统化（sideband 牢、guidance 灵活）？这条假设是 thesis 把 inversion-based / attention-injection / control-injection 三类编辑统一在 [[wiki/concepts/sideband-conditioning|sideband 抽象]]下后必须回答的核心问题。
>
> 3. **采样速度是开放赛道** —— ✅ **DDIM 已部分验证**。[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]] 证明加速可以是**纯采样期**的事：不重训、不动 backbone，只把前向从马尔可夫松绑为非马尔可夫族（[[wiki/concepts/non-markovian-diffusion]]），就能在子序列上跳步、10–50× 加速。这把"采样加速"从训练问题降级为推断问题——对本就要反复采样的编辑场景尤其关键。进一步地，DDIM 的确定性采样（$\sigma=0$）与 ODE 反向积分给出了 **inversion** 的理论入口，直接服务于 overview「主要派系」中的 inversion-based 一类。**Score SDE 把这一切收进连续化总纲**：DDIM 的确定性采样正是 [[wiki/concepts/probability-flow-ode|probability-flow ODE]] 的离散特例，而 PF-ODE 在同一框架里统一了"确定性快采样 + 精确似然 + 可逆 inversion"（[[wiki/sources/songScoreBasedGenerativeModeling2021|Song et al. 2021]]）。✅ **[[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 再加一证**：[[wiki/concepts/optimal-transport-path|OT 直线路径]]把采样 NFE 降到约 60%、且采样成本在训练期**恒定**（不像 score matching 越训越贵）——加速可以来自"路径设计"本身，而非只靠采样器/蒸馏。✅ **[[wiki/sources/liuFlowStraightFast2022a|Rectified Flow]] 把这条线推到极限**：[[wiki/concepts/reflow|reflow]] 在**训练阶段**迭代把 ODE 拉直——直线度单调升、凸传输代价单调降，极限直线时**单步 Euler 即精确**。加速来源由此完整化为一条层级：**采样器（DDIM 跳步）→ 路径设计（FM-OT）→ 训练阶段轨迹改造（RF reflow）→ 蒸馏（1-step generator）**——每一层都对应"动训练目标"的不同程度。仍待 ingest：Consistency Models（Yang Song）/ SD3 / FLUX / RF-Inversion 对**编辑质量**的影响。
>
> 推论 3 同时**强化了推论 1 的可变性光谱**：DDIM 动的是"采样链的形状（步数、$\sigma$）"，属于光谱中"研究杠杆 / 介入方式"一档，且代价极低——是"范式不变、组件可调"的又一个干净样本。
>
> 4. **transport 是比 denoising 更高一层的候选不变量；bridge-SDE 侧欠发达 = thesis 施力点**（v0.5 新增，[[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] ingest + 用户定向押注）
>
>    [[wiki/sources/zhouDenoisingDiffusionBridge2023|DDBM]] 把范式从"iterative denoising（noise→data）+ 注入条件"**泛化**为"两个任意端点分布之间、由学到的 drift/score 驱动的 **iterative transport**"——editing/translation 是 paired transport 的特例、denoising 是噪声端点的特例。这给推论 1 的"近乎不变范式"提了一个更高层的候选表述。
>
>    新增一条与可变性光谱**正交**的结构轴：**用什么动力学 realize transport**——随机 **bridge SDE**（[[wiki/methods/ddbm|DDBM]]，学 score、[[wiki/concepts/doob-h-transform|Doob h]] 钉端点）vs 确定 **bridge ODE**（[[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]]，学速度场）。二者经 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]] 统一（SDE↔ODE 可互转），故这是 realization 选择而非本质分界（见 [[wiki/concepts/diffusion-bridge]]）。
>
>    **thesis 押注**：bridge-**ODE** 侧 inversion/editing 已成熟（RF-Inversion）；bridge-**SDE** 侧因随机项 inversion 闭合困难、基本空白——发展 bridge-SDE inversion/editing 为本 thesis 选定施力点（[[research/ideas]] 2026-06-01 条、[[research/thesis]] v0.1）。
>
>    ⚠️ **诚实 caveat**：(i)"SDE/ODE 双构造"lens 源自 [[wiki/concepts/stochastic-interpolants|stochastic interpolants]]，**非原创**，novelty 须落在 SDE 侧具体方法；(ii) DDBM 是 **translation** 非 text-editing，"editing = paired transport"目前是**泛化假设、未验证**；(iii) bridge-SDE 已有 DDBM/I²SB/SB/BBDM，**非 greenfield**，开题前须 sweep（尤其 2024 加速类工作）。
>
>    🔁 **2026-06-24 方向复审（与 [[research/thesis]] / [[research/ideas]] 同步）**：caveat (iii) 的 sweep 已做——bridge-SDE editing 全家桶**三次 sweep 三撞全占**（见 [[wiki/synthesis/bridge-sde-editing-landscape]]），bridge-SDE 押注**暂挂**。当前活线已转向 **energy-guidance**（discriminative logits → energy → guidance，师兄在研线）：sweep 同样显示 generic 形式红海，但坐导师在研线、**sliver 已定**（[[wiki/methods/freedom|FreeDoM]] 三段改进 × 收束到 flow 底座），详见 [[wiki/synthesis/energy-guidance-landscape]] 与 [[research/ideas]]。**working thesis 版本号仍不动**，待第一个实验出结果再定升级。

> _下次 ingest 后请重新审视上述推论是否仍然成立（**推论 4 为 2026-06-01 新增的方向级押注，尚待 editing 任务验证**）。推论 1、3 已被 DDIM + [[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]] + [[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] + [[wiki/sources/liuFlowStraightFast2022a|Rectified Flow]] + [[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] + [[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] 正向支持（FM 是"换训练目标却不离范式"的最强样本；RF 进一步把加速来源从"路径设计"推到"训练阶段轨迹改造"；**LDM 给"组件可新增而非替换"补一档"压缩层"**；**ControlNet 把"组件可新增"推到 sideband 极端——不动 backbone 参数、只克隆结构外挂**，并催生统一抽象 [[wiki/concepts/sideband-conditioning|frozen backbone + trainable sideband]] 把四类编辑方法重新组织；**两条演化线在 SD3/FLUX 处汇合**）；推论 2 已从 Score SDE 拿到形式化抓手（逐 $t$ 的条件 score 引导项），并被 FM 追加"跨范式时间坐标须对齐"的 caveat、被 LDM 追加"autoencoder 重建上限作为正交 fidelity 上界"的新维度、被 ControlNet 追加"全 $t$ sideband vs 逐 $t$ guidance"的对照 caveat，**并被 [[wiki/sources/mengSDEditGuidedImage2022|SDEdit]] 首次直接编辑实证**（$t_0$ = [[wiki/concepts/noising-strength|noising strength]] 旋钮，Fig 3 单调 realism↔faithfulness 曲线）——"高 $t$ 改结构、低 $t$ 改细节"从假设升级为可测量；**仍待**：text/attention 类编辑（Prompt-to-Prompt 等）是否给出与 noising-based 一致的 $t$ 依赖。RF 还引入新研究变量 [[wiki/concepts/transport-coupling|coupling]]——DDIM inversion 的稳定性可被重表述为 coupling rewiring 的稳定性，是 thesis 可用的诊断语言。_

## 当前关注的子问题

- 编辑保真度（fidelity）vs 可控性（controllability）trade-off
- 全局编辑 vs 局部编辑的统一框架
- [[wiki/concepts/flow-matching|Flow Matching]] / [[wiki/methods/rectified-flow|Rectified Flow]] 相对 score-based diffusion 在编辑任务上的优劣
- Inversion 质量对编辑结果的影响
- 评测：CLIP-based 指标的局限与替代

## 主要派系（待填充链接）

> 🆕 [2026-05-27] 总前提（[[wiki/sources/rombachHighResolutionImageSynthesis2022|LDM]] ingest 后补充）：以下派系**多数默认建立在 [[wiki/methods/ldm|LDM]] / [[wiki/entities/stable-diffusion|Stable Diffusion]] 的 latent 管线之上**——这些方法的 inversion / attention / sideband 操作都发生在 $z=\mathcal E(x)$ 上，pixel 空间仅出现于 input/output。这是 thesis 在做 fidelity 评测时必须显式建模的前提（见推论 2 "正交维度" 注）。**⚠️ 例外（[[wiki/sources/mengSDEditGuidedImage2022|SDEdit]] ingest 后补充 2026-05-29）**：SDEdit（2021.08）**早于 SD、底座无关**——它套在任意预训练 [[wiki/concepts/score-sde|score model]] 上（pixel 或 latent 皆可）。所以"派系建立在 SD 上"是**经验常态而非定义**：noising/inversion 这类**采样期**方法本质底座无关，只是后来大家都拿 SD 当现成 prior 而已。
>
> 🆕 [2026-05-28] 统一上层抽象（[[wiki/sources/zhangAddingConditionalControl2023|ControlNet]] ingest 后补充）：以下**五类的前四类**都可重新归到 [[wiki/concepts/sideband-conditioning|frozen backbone + trainable sideband]] 这一抽象之下——差别只在 sideband 接入主干的位置（初始噪声 / attention map / skip connection / 训练 token）与训练成本（zero-shot / 优化几百步 / 50k–200k 训练）。flow-matching-based 是底座层面的变化（换训练目标），与前四类正交。

- **Inversion / noising-based（🆕 [2026-05-29] 改名：SDEdit ingest 后纳入 noising 一支）**：✅ [[wiki/methods/sdedit|SDEdit]]（已 ingest，最朴素：**不优化不反演**，直接把 guide 加噪到 $t_0$ 当起点）；待 ingest：DDIM-inversion、Null-text inversion、Negative-prompt inversion、… —— sideband = "在初始 noisy 状态上设置/优化的扰动"，按成本递增排一条光谱：**SDEdit（零成本加噪）→ DDIM-inversion（确定性反演）→ Null-text（优化几百步）**；在 LDM/SD 上都在 latent 往返、fidelity 上界已被 autoencoder 重建误差污染。核心旋钮 [[wiki/concepts/noising-strength|noising strength $t_0$]]
- **Instruction-tuned**：InstructPix2Pix、MagicBrush、… —— sideband = "在 SD 上做 instruction-following 全模型微调"；与其他派系不同，**改 backbone 参数**（仍待 ingest）
- **Attention-injection**：Prompt-to-Prompt、Plug-and-Play、MasaCtrl、Attend-and-Excite、StyleAligned、… —— sideband = "在 cross/self-attention map 上加性 / 替换的修改"；直接攻击 LDM 的 [[wiki/concepts/cross-attention|cross-attention map]]（仍待 ingest）
- **Control/sideband-injection（🆕 [2026-05-28] LDM ingest 后新增第 5 类）**：✅ [[wiki/methods/controlnet|ControlNet]]（已 ingest）、T2I-Adapter、IP-Adapter、GLIGEN、… —— sideband = "在 skip connection 上加性的空间对齐条件残差"；用 [[wiki/concepts/zero-convolution|zero conv]] 等初始化恒等机制做稳定 fine-tune；与 attention-injection **正交**（一个在 U-Net 外部 sideband、一个在 U-Net 内部 attention map），是 thesis 在资源约束下最可行的研究通道
- **Flow-matching-based**：✅ [[wiki/methods/flux-kontext|FLUX.1 Kontext]]（已 ingest，BFL 2025；该派系首篇）、SD3 / FLUX 基础模型 / RF-Inversion、… —— [[wiki/methods/ldm|LDM]] 压缩层 ⊕ [[wiki/methods/rectified-flow|RF]]/[[wiki/concepts/flow-matching|FM]] 训练目标的工业实现；与前四类**正交**——底座换了，前四类的 sideband 接口需重设计。**🆕 Kontext 还引入第六条条件注入通道 [[wiki/concepts/in-context-conditioning|in-context token 序列拼接]]**（区别于 noising/cross-attn/attention-injection/sideband）：把编辑建成条件生成 $p(x\mid y,c)$ 而非显式 bridge——印证编辑主线在 in-context 条件 FM、不在 bridge（仍待 ingest：SD3 / FLUX 基础模型原文）

## 关键源

```dataview
LIST title
FROM "wiki/sources"
SORT updated DESC
LIMIT 20
```

## 待调研方向

> 由 [2026-05-14] lint 填充。来源：现有 wiki 页中反复出现但尚无独立条目 / 标注"待补"的术语。

- **score-based 主线的两个上游**：[[wiki/methods/diffusion-2015|Sohl-Dickstein 2015]]（diffusion 原型）与 [[wiki/methods/ncsn|NCSN (Song & Ermon 2019)]] —— 已建 stub，待 ingest 原文厘清与 DDPM 的精确差异
- ✅ **连续时间统一框架（已 ingest 2026-05-20）**：[[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE (Song et al. 2021)]] —— 已严格化"small-$\beta$ ⇒ 反向高斯"的离散近似，打通 probability-flow ODE → DDIM 脉络；新增 [[wiki/concepts/fokker-planck-equation]] / [[wiki/concepts/probability-flow-ode]] / [[wiki/concepts/predictor-corrector-sampling]] 三页，并把 [[wiki/concepts/score-sde]] 升级为完整主条目。**仍开放**：VP/VE/sub-VP 在编辑任务上的优劣（与 Flow Matching 的精确关系已由 2026-05-24 ingest 的 [[wiki/sources/lipmanFlowMatchingGenerative2023|FM]] 厘清，见下条）
- **guidance 旋钮**：[[wiki/concepts/classifier-free-guidance|Classifier-Free Guidance]] 及其前身 [[wiki/concepts/classifier-guidance|classifier guidance]]（Dhariwal & Nichol 2021）—— 编辑层四旋钮之一；classifier guidance 已借 Score SDE 的贝叶斯链成页，两者原文（Dhariwal & Nichol 2021 / Ho & Salimans 2022）仍待 ingest
- **采样加速线**：[[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]]、[[wiki/sources/songScoreBasedGenerativeModeling2021|Score SDE]]、[[wiki/sources/lipmanFlowMatchingGenerative2023|Flow Matching]] 与 [[wiki/sources/liuFlowStraightFast2022a|Rectified Flow]] 已 ingest（验证推论 3——DDIM 给离散跳步，Score SDE 给 [[wiki/concepts/probability-flow-ode|PF-ODE]] 连续总纲 + PC 校正，FM 给 [[wiki/concepts/optimal-transport-path|OT 路径]]级的 NFE 下降，RF 给 [[wiki/concepts/reflow|reflow]] 训练阶段直线化 + 1-step 蒸馏入口）；下一步：Consistency Models（Yang Song 后续）、SD3 / FLUX（RF 工业实现，填 overview「主要派系→flow-matching-based」）、RF-Inversion 系（RF backbone 上的编辑方法，直接对接 thesis）—— 这些对**编辑质量**的影响仍是空白
- **训练目标的"调参级"修改**：v-prediction（Salimans & Ho 2022）、IDDPM 的 $\Sigma_\theta$ 学习、Min-SNR / EDM preconditioning —— 可作小幅 ablation，暂未值得单独成页
- ✅ **编辑方法族的共同底座（2026-05-27 LDM ingest）**：五类派系全部默认建立在 [[wiki/methods/ldm|LDM]] / [[wiki/entities/stable-diffusion|Stable Diffusion]] 之上——已在「主要派系」节加总前提注。
- ✅ **Control/sideband-injection 首篇（2026-05-28 ControlNet ingest）**：[[wiki/methods/controlnet|ControlNet]] 已 ingest，新增第 5 类派系；同时催生统一抽象 [[wiki/concepts/sideband-conditioning|frozen backbone + trainable sideband]]，把前四类编辑方法重新组织在该抽象下。
- ✅ **Inversion / noising-based 首篇（2026-05-29 SDEdit ingest）**：[[wiki/methods/sdedit|SDEdit]] 已 ingest——该派系奠基且最朴素的一支（不优化、不反演，纯加噪 + reverse）。建 [[wiki/concepts/noising-strength|noising strength $t_0$]] 概念页（= SD img2img `strength`），是推论 2「在哪个 $t$ 介入」的首个直接实证。**仍待 ingest** 同派系更精细者：DDIM-inversion / Null-text inversion（带优化，忠实度更高、成本更高）；以及 Attention-injection 首篇（推荐 Prompt-to-Prompt）/ Instruction-tuned 首篇（InstructPix2Pix）
- **DDIM inversion**：DDIM 的 ODE 可反向积分把 $x_0$ 编码回 $x_T$，已在 [[wiki/sources/songDenoisingDiffusionImplicit2022|DDIM]]、[[wiki/methods/ddim]]、[[wiki/concepts/non-markovian-diffusion]] 多处被提及，是 inversion-based 编辑的技术底座——暂不单独成页，待第一篇用它的编辑论文 ingest 时再建 `concepts/` 或 `methods/` 条目。**注**（2026-05-27）：在 LDM/SD 上 DDIM inversion **都在 latent 上做往返**，其失败模式可能与 [[wiki/concepts/perceptual-compression|autoencoder 重建误差]]纠缠——thesis 可立此为诊断维度
- ✅ **DDIM ↔ Flow Matching ↔ Rectified Flow 的关系（2026-05-24 / 2026-05-26 两次 ingest 正式化）**：DDIM 的确定性采样（$\sigma=0$）本身就是解一个 ODE（$d\bar x=\varepsilon_\theta\,d\sigma$，即 [[wiki/concepts/probability-flow-ode|probability-flow ODE]]），所以 DDIM 已有"flow 味"。区别在于：DDIM/PF-ODE 的 ODE 是从**已训好的 ε/score 网络事后导出**，不动训练；[[wiki/concepts/flow-matching|Flow Matching]] 则**直接把训练目标换成回归速度场**（[[wiki/concepts/conditional-flow-matching|CFM]]），ODE 是训出来的本体；[[wiki/methods/rectified-flow|Rectified Flow]] 在 FM 之上加 [[wiki/concepts/reflow|reflow]]——把训出来的 ODE 当作新 coupling 再训一遍，递推拉直。一句话脉络：**DDIM = diffusion 的训练 + flow 的采样；FM = 连训练也 flow 化；RF = FM + 把 ODE 在训练阶段反复拉直**。精确公式关系：RF 的线性插值 = FM-OT 路径取 $\sigma_{\min}=0$；RF 的额外卖点是任意 coupling 接口（[[wiki/concepts/transport-coupling]]）+ reflow。已建 [[wiki/concepts/reflow]] / [[wiki/concepts/transport-coupling]] 两页。**仍待 ingest**：SD3（Esser et al. 2024）/ FLUX / RF-Inversion 系。
- 🆕 **Bridge 线（2026-06-01 DDBM ingest + 用户定向，推论 4 的载体）**：[[wiki/methods/ddbm|DDBM]] 已 ingest，建 [[wiki/concepts/diffusion-bridge]] / [[wiki/concepts/doob-h-transform]] / [[wiki/concepts/infinitesimal-generator]] / [[wiki/concepts/stochastic-interpolants]]，并 ingest [[wiki/methods/dbim|DBIM]]（DDBM 的 DDIM 化 + 可逆桥）与 [[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants]]（统一框架）。**thesis 选定方向 = bridge-SDE 上的 editing**（[[research/ideas]] / [[research/thesis]]；全景地图 [[wiki/synthesis/bridge-sde-editing-landscape]]）。**已验明 KILL**：纯"bridge-SDE 理论 / 脱离 diffusion"撞 [[wiki/sources/albergoStochasticInterpolants2023|Stochastic Interpolants]]（理论侧已满）。**仍待 ingest（按 thesis 距离）**：I²SB（tractable Schrödinger Bridge）/ Schrödinger Bridge·Bridge-Matching（Shi 2023）/ BBDM / EDM（Karras 2022）。
