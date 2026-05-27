# Log

> Append-only 时间线。每条 entry 以 `## [YYYY-MM-DD] <op> | <subject>` 起始，便于 `grep "^## \[" log.md | tail`。

## [2026-05-27] lint | LDM ingest 后整改：5 处 Stale 关闭
- 扫描：56 wiki 页 · 全部 wikilink 解析 · broken **0**（grep 报告 17 条均误报：14 条 markdown-table-pipe-escape、3 条 raw/assets/ 下 .webp 嵌入）/ orphan **0** / weakly-linked **0** / frontmatter 缺失 **0** / 矛盾 **0**
- updated: `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md` —— S1（line 108）"Stable Diffusion / Latent Diffusion 等" 裸文本 → `[[wiki/methods/ldm|Stable Diffusion / LDM]] (Rombach et al. 2022)`
- updated: `wiki/methods/ddpm.md` —— S2（line 74）"Stable Diffusion / LDM" 裸文本 → `[[wiki/methods/ldm|LDM (Stable Diffusion)]]`
- updated: `wiki/concepts/classifier-free-guidance.md` —— S3（line 26）"Stable Diffusion" 裸文本 → `[[wiki/methods/ldm|Stable Diffusion]]`
- updated: `wiki/concepts/epsilon-parameterization.md` —— S4（line 51）"Stable Diffusion" 裸文本 → `[[wiki/methods/ldm|Stable Diffusion]]`
- updated: `wiki/overview.md` —— S5（line 38）"cross-attention 在 U-Net 里" → `[[wiki/concepts/cross-attention|cross-attention]] 在 U-Net 里`（推论 1 关键修正(iii) 的精确锚点）
- 五页 `updated` 同步刷至 2026-05-27（overview 已在前一日 ingest 时刷过）
- Missing 候选（频繁被提及但未建页）已识别但本次不动，按 thesis 距离排序：
  - **P0**：第一篇 text-guided editing 论文（推荐 Prompt-to-Prompt → attention-injection 派系直接攻击 cross-attention）；SD3（Esser et al. 2024）；FLUX
  - **P1**：VQGAN（Esser, Rombach & Ommer 2021）—— LDM 上游；ControlNet
  - **P2**：classifier guidance / CFG 原文
  - **P3**：Sohl-Dickstein 2015 / NCSN 原文（已有 stub）；InstaFlow / Stochastic Interpolants
  - **schema-defer**：DDIM inversion 独立 concept 页（overview「待调研方向」已规定待第一篇 editing 论文 ingest 时再建）
- 仍开放（Suggestion，待用户定）：下一轮 ingest 候选——Prompt-to-Prompt 或 SD3 二选一作起点

## [2026-05-27] ingest | High-Resolution Image Synthesis with Latent Diffusion Models (Rombach et al. 2022)
- created: `wiki/sources/rombachHighResolutionImageSynthesis2022.md` —— LDM / Stable Diffusion 方法论原点
- created: `wiki/methods/ldm.md` —— Latent Diffusion 方法主页（family=other，与 ddpm/ddim 同档的"底座"类）
- created: `wiki/concepts/cross-attention.md` —— token 类条件注入接口；列出与 self-attn / conditional drift / CFG / classifier guidance 的关系，及在 attention-injection 编辑方法族中的核心地位
- created: `wiki/concepts/perceptual-compression.md` —— LPIPS+adv+recon+KL/VQ-reg 的 autoencoder 训练；明确 hard upper bound $\mathcal D(\mathcal E(x))\neq x$ 对 thesis 的含义
- created: `wiki/concepts/latent-space-generative-modeling.md` —— 抽象范式页；区分与 VAE / pixel diffusion；指明 SD3/FLUX = LDM 压缩层 ⊕ FM/RF 训练目标
- created: `wiki/entities/robin-rombach.md` / `bjorn-ommer.md` / `compvis.md` / `lmu-munich.md` —— 一作 / 通信作者 / 实验室 / 机构
- updated: `wiki/overview.md` —— **不升 working thesis 版本号，轻量更新**：(1) 可变性光谱"可演化但非主战场"档新增「压缩层」组件，附独立注释说明其正交于其他三档；(2) 推论 1 加 LDM 作为「组件可新增而非替换」的样本，并指出 SD3/FLUX 是两条正交演化线（LDM 压缩层 + FM/RF 训练目标）的汇合点；(3) 推论 2 加「正交维度」注：autoencoder 重建误差是 fidelity 的 hard upper bound、独立于 $t$ 与 guidance 强度；(4) 「主要派系」节加总前提注——四类全部默认建立在 LDM 上，inversion-based 显式注明"latent 上往返"，attention-injection 显式注明"攻击 cross-attention map"；(5) 「重审注」与「待调研方向」吸收 LDM；sources 加 [[wiki/sources/rombachHighResolutionImageSynthesis2022]]；updated 2026-05-27
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/rombachHighResolutionImageSynthesis2022.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（§5.1 例外）
- thesis：working thesis v0.4 不变（LDM 没动 paradigm、没动训练目标，是"组件维度"上的新增；与 FM/RF 正交，待 SD3/FLUX ingest 时一并考虑是否升 v0.5）
- 用户决定：将"autoencoder 重建上限会限制编辑保真度"按 🟣 级 thesis-implication 处理（用户原 annotation 全用黄色，未打 🟣/🔵）；该 implication 已落入 source 页「对我的 thesis 的启示」与 overview 推论 2「正交维度」注
- 待 ingest（按 P0 → P1 排）：第一篇 text-guided editing 论文（建议 Prompt-to-Prompt 或 InstructPix2Pix，直接对接 thesis）；SD3 / FLUX / RF-Inversion；VQGAN 原文（LDM 上游）

## [2026-05-26] lint | RF ingest 后整改：4 处 Stale 关闭
- 扫描：47 wiki 页 · 562+ wikilink · broken **0** / orphan **0** / weak **0** / frontmatter **0**
- updated: `wiki/sources/lipmanFlowMatchingGenerative2023.md` —— S1（line 113）"Liu et al. 2022 ...为并行工作（待 ingest 原文）"→ ✅ 已 ingest，改链至 [[wiki/sources/liuFlowStraightFast2022a]] 并指 method 页异同表；S2（line 127）open question 中 Liu 2022 部分关闭，仅 SD3/FLUX 待 ingest
- updated: `wiki/sources/songScoreBasedGenerativeModeling2021.md` —— S3（line 107）open question "与 RF 关系（待 ingest）"→ ✅，加 PF-ODE vs RF 的"事后导出 vs 主动拉直"刻画
- updated: `wiki/sources/songDenoisingDiffusionImplicit2022.md` —— S4（line 98 下游节）补 RF 反链，串起"DDIM → PF-ODE → RF/reflow"确定性 ODE 家谱
- 三页 `updated` 刷新至 2026-05-26
- Missing 候选（高频但未建页）已识别但本次不动：P0 = SD3 / RF-Inversion（直接对接 thesis）；P1 = FLUX / Consistency Models；P2 = Stochastic Interpolants / InstaFlow（已记在 overview「待调研方向」与各 source 的 open questions）
- 仍开放（Suggestion，待用户定）：是否 ingest SD3 原文以填 overview「主要派系→flow-matching-based」首条；是否 ingest RF-Inversion 第一篇编辑论文

## [2026-05-26] ingest | Flow Straight and Fast: Rectified Flow (Liu, Gong & Liu 2022)
- created: `wiki/sources/liuFlowStraightFast2022a.md`
- created: `wiki/concepts/reflow.md`、`wiki/concepts/transport-coupling.md`
- created: `wiki/entities/xingchao-liu.md`、`wiki/entities/qiang-liu.md`
- updated: `wiki/methods/rectified-flow.md` —— stub → 完整方法页（核心算法、与 FM/OT 路径精确异同表、关系网；status draft → active；sources 加 Liu 2022）
- updated: `wiki/concepts/flow-matching.md` —— 并行工作处把 Liu 2022 从"待 ingest"标 ✅，注明"RF = FM-OT 取 $\sigma_{\min}=0$ + 任意 coupling + reflow"
- updated: `wiki/concepts/optimal-transport-path.md` —— 下游 RF 改 wikilink，精确刻画与 OT 路径的边界关系；sources 加 Liu 2022
- updated: `wiki/concepts/probability-flow-ode.md` —— "PF-ODE / FM / RF" 三家谱厘清：PF-ODE 事后导出 vs RF 主动拉直；sources 加 Liu 2022
- updated: `wiki/overview.md` —— 轻量更新（不升版本）：推论 3 加 RF/reflow 把加速来源完整化为四层（采样器→路径→训练阶段轨迹→蒸馏）；推论末「重审注」吸收 RF；待调研方向「DDIM↔FM↔RF」三角厘清，RF 标 ✅，新增 SD3/FLUX/RF-Inversion 待 ingest
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/liuFlowStraightFast2022a.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`、`updated`（§5.1 例外）
- thesis：working thesis v0.4 不变（按用户选择"轻量更新不升版本"）；RF 作为 FM 推论 3 的强化验证而非范式断点

## [2026-05-24] lint | FM ingest 后整改：消 stale、retarget pf-ode 链、建 Rectified Flow stub
- 扫描：562 条 wikilink —— broken **0** / orphan **0**（最弱 imagenet 仅 1 入链，已 polish 补强）；frontmatter 无缺字段
- created: `wiki/methods/rectified-flow.md` —— Missing 项 stub（RF 在 6+ 页被提及却无独立页），作为 FM 并行/下游线与多条「待 ingest」的落点
- updated: `wiki/sources/songScoreBasedGenerativeModeling2021.md` —— Stale：Open question「与 FM 精确关系 待 ingest」标 ✅ 已由本次 ingest 解决，改链 flow-matching / RF stub；刷新 updated
- updated: `wiki/concepts/non-markovian-diffusion.md` —— Mis-link：「probability-flow ODE」从 `score-sde` 改指专页 `probability-flow-ode`；刷新 updated
- updated: `wiki/overview.md` —— 当前子问题 + 待调研方向里纯文字 "Flow Matching / Rectified Flow" 挂链（RF 指向新 stub）
- updated: `wiki/concepts/optimal-transport-path.md`、`wiki/concepts/flow-matching.md`、`wiki/sources/lipmanFlowMatchingGenerative2023.md` —— "Rectified Flow" 纯文字 → 链到 RF stub
- updated: `wiki/benchmarks/cifar10.md`、`wiki/benchmarks/metrics.md` —— polish：补 imagenet 姊妹/数据集互链（消除 imagenet 弱连通）
- updated: `wiki/methods/ddim.md` —— polish：下游补 [[wiki/concepts/flow-matching]] / [[wiki/methods/rectified-flow]] 反链；刷新 updated
- 矛盾：**无**（FM 时间反向约定在各新页内部一致）
- 仍开放（Suggestion，待用户定）：ingest Rectified Flow → SD3 / FLUX（填 overview「主要派系→flow-matching-based」）；Consistency Models、Neural ODE / Chen 2018 列为将来 Missing 候选

## [2026-05-24] thesis-update | overview working thesis v0.3 → v0.4（吸收 Flow Matching）
- updated: `wiki/overview.md` —— 用户批准「全部应用」：
  - 标题 v0.3 → v0.4（+ Flow Matching）；frontmatter 加 FM source
  - 推论 1：FM 证据从"待验证"升级为 ✅「最强样本」——真换训练目标（CFM 回归速度场）却不离范式，同架构 FM-OT 三项超 DDPM
  - 推论 2：新增跨范式 caveat——FM 的 $t$（噪声→数据插值系数）与 diffusion 方向相反，比较"介入时间步"须先对齐坐标
  - 推论 3：加 FM 一证——OT 路径把 NFE 降到约 60%、训练期采样成本恒定，加速可来自"路径设计"
  - thesis 末「重审注」同步；待调研方向两处「待 ingest FM 原文」标 ✅ 并改指新建 FM 概念页

## [2026-05-24] ingest | Flow Matching for Generative Modeling (Lipman et al. 2023)
- created: `wiki/sources/lipmanFlowMatchingGenerative2023.md`
- created: `wiki/concepts/flow-matching.md`、`wiki/concepts/conditional-flow-matching.md`、`wiki/concepts/continuous-normalizing-flow.md`、`wiki/concepts/optimal-transport-path.md`
- created: `wiki/entities/yaron-lipman.md`、`wiki/entities/ricky-chen.md`、`wiki/entities/meta-ai-fair.md`
- created: `wiki/benchmarks/imagenet.md`
- updated: `wiki/benchmarks/cifar10.md` —— 新增 FM 同架构消融表（NLL/FID/NFE，附"未为 CIFAR 调优"caveat）+ 回填源
- updated: `wiki/concepts/score-sde.md`、`wiki/concepts/probability-flow-ode.md` —— 区分 FM（直接训速度场）vs PF-ODE（事后从 score 导出）；纯文字 "Flow Matching" 改 wikilink
- updated: `wiki/concepts/score-matching.md` —— 增 CFM≡DSM 的 vector-field 类比（Theorem 2）
- updated: `wiki/methods/ddpm.md` —— 关联补「diffusion 高斯路径是 FM Gaussian 路径族特例」
- updated: `index.md` —— 刷新 updated（新页由 Dataview 自动收录）
- updated: `raw/literature-notes/lipmanFlowMatchingGenerative2023.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（§5.1 例外）
- thesis：`wiki/overview.md` working thesis v0.3 → v0.4 已经用户批准应用，单列于上方 thesis-update 条目

## [2026-05-20] refactor | 补 Sohl-Dickstein entity（lint Missing 的漏网项）
- created: `wiki/entities/jascha-sohl-dickstein.md` —— diffusion 鼻祖，wiki 中 13 次提及却无 entity 页；入链自 diffusion-2015（method 页点名却无反链）与 Score SDE source
- updated: `wiki/methods/diffusion-2015.md` —— 关联区加人物链接
- updated: `wiki/sources/songScoreBasedGenerativeModeling2021.md` —— 作者人物链补 Sohl-Dickstein
- 评估结论：methods/ 无缺口（Score SDE 是框架→入 concepts/，VP/VE 离散化已标注于 ddpm/ncsn）；Kingma(2 次)/Google Brain(2 次) 频率偏低暂不建

## [2026-05-20] lint | Score SDE ingest 后整改：消 stale、统一记号与 frontmatter
- 扫描结果：无孤立页（每页 ≥3 入链）、无真失效 wikilink
- updated: `wiki/sources/songDenoisingDiffusionImplicit2022.md` —— S1：关闭"待 ingest Yang Song 2021"open question（PF-ODE 对应已建立）
- updated: `wiki/entities/uc-berkeley.md` —— S2：删除把 Score SDE 误挂到 Berkeley 的待补项（作者属 Stanford + Google Brain）
- updated: `wiki/concepts/langevin-dynamics.md` —— C1：补 Langevin 步长记号桥接注（$\varepsilon=\eta/2$）
- updated: `wiki/sources/songScoreBasedGenerativeModeling2021.md` —— F1：`raw:` 字段列表→字符串，与另两篇 source 统一
- updated: `wiki/methods/ncsn.md` —— F2：`sources: []` 回填 Score SDE（正文已引用）
- created: `wiki/concepts/classifier-guidance.md` —— M1（用户选"建独立页"）：贝叶斯链来自 Score SDE，含与 CFG 对比表；入链自 CFG / score-sde / overview
- created: `wiki/benchmarks/metrics.md` —— M2（用户选"建汇总页"）：IS / FID / bits-per-dim 定义+方向+缺陷；入链自 cifar10 / lsun
- updated: `wiki/concepts/classifier-free-guidance.md`、`wiki/concepts/score-sde.md`、`wiki/overview.md` —— 挂入 classifier-guidance 链接
- updated: `wiki/benchmarks/cifar10.md` —— 补 Score SDE 结果行（IS 9.89 / FID 2.20 / 2.99 bits/dim）+ 链接 metrics 页；`wiki/benchmarks/lsun.md` 链接 metrics 页
- 仍开放（Suggestion，待用户定）：ingest NCSN 原文 / Flow Matching / Consistency Models

## [2026-05-20] ingest | Score-Based Generative Modeling through Stochastic Differential Equations (Song et al. 2021)
- created: `wiki/sources/songScoreBasedGenerativeModeling2021.md`
- created: `wiki/concepts/fokker-planck-equation.md`
- created: `wiki/concepts/probability-flow-ode.md`
- created: `wiki/concepts/predictor-corrector-sampling.md`
- created: `wiki/entities/yang-song.md`
- updated: `wiki/concepts/score-sde.md` —— 由 stub 升级为完整主条目（VP/VE/sub-VP、训练目标、PC/ODE 采样、条件生成）
- updated: `wiki/concepts/score-matching.md` —— 增「条件 score 监督 ⇒ 边缘 score」恒等式与 loss 展开、连续时间 DSM
- updated: `wiki/concepts/langevin-dynamics.md` —— 增 Langevin 作 PC corrector
- updated: `wiki/concepts/classifier-free-guidance.md` —— 增条件反向 SDE（贝叶斯/classifier guidance）连续底座
- updated: `wiki/concepts/diffusion-process.md` —— 连续时间 SDE 极限 open question 标记完成
- updated: `wiki/methods/ncsn.md` —— 标注 NCSN = VE-SDE 离散化
- updated: `wiki/methods/ddpm.md` —— 标注 DDPM = VP-SDE 离散化
- updated: `wiki/methods/ddim.md` —— DDIM 确定性采样 = probability-flow ODE 离散化，链接新页
- updated: `wiki/entities/stefano-ermon.md`、`wiki/entities/stanford.md` —— 回填 Score SDE 工作与 Yang Song 链接
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/songScoreBasedGenerativeModeling2021.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（re-import 重置后二次回填，§5.1 例外）

## [2026-05-20] thesis-update | overview working thesis v0.2 → v0.3（吸收 Score SDE）
- updated: `wiki/overview.md` —— 用户批准「大胆改」：
  - 标题 v0.2 → v0.3（DDPM + DDIM + Score SDE）；frontmatter 加 source
  - thesis 正文增「Score SDE 把统一从类比抬升为严格连续时间理论 + 同一 score 网络可换三种采样器」
  - 推论 1 增「一网络三采样器」干净样本，光谱细化为 训练目标 > backbone > 采样器/guidance/介入时间步
  - 推论 2 增「条件反向 SDE 逐 $t$ 引导项」作为 fidelity↔controllability 旋钮的形式化抓手
  - 推论 3 增 probability-flow ODE 连续化总纲
  - 待调研方向：Score SDE 标 ✅ 已 ingest，采样加速线补 PF-ODE/PC

## [2026-05-14] lint | DDIM ingest 后整改：修作者实体错挂、补 entity 页、消歧 "Song et al. 2021"、回填 benchmark
- created: `wiki/entities/jiaming-song.md`、`wiki/entities/stefano-ermon.md`、`wiki/entities/stanford.md` —— M1：DDIM 作者实体页（此前实体覆盖与 DDPM 不对称）
- updated: `wiki/sources/songDenoisingDiffusionImplicit2022.md` —— C1：实体节错挂 [[wiki/entities/jonathan-ho]]（非 DDIM 作者）→ 改挂正确作者；C2：补注 Zotero year 2022 vs 正式 ICLR 2021；M2：关系节补「基准」链接；S1：alias "Song et al. 2021" → "Song, Meng & Ermon 2021"
- updated: `wiki/concepts/score-sde.md` —— S1：消歧 "Song" = Yang Song（≠ DDIM 的 Jiaming Song），alias 与正文同步
- updated: `wiki/concepts/score-matching.md` —— S1：Score SDE 引用改 "Yang Song et al. 2021"
- updated: `wiki/concepts/langevin-dynamics.md` —— B1：纯文本 "DDIM" → wikilink；补 sources 字段
- updated: `wiki/concepts/epsilon-parameterization.md` —— B2：参数化表中 "DDIM" 改 wikilink 并修正措辞（DDIM 仍用 ε 网络，只是更新式经由预测 $x_0$ 表达）
- updated: `wiki/benchmarks/cifar10.md`、`wiki/benchmarks/lsun.md` —— M2：回填 DDIM 分步数 FID 行 / 定性结果与 sources、出处字段
- updated: `wiki/overview.md` —— S2：「待调研方向」新增 DDIM inversion 条目

## [2026-05-14] ingest | Denoising Diffusion Implicit Models (Song et al. 2021)
- created: `wiki/sources/songDenoisingDiffusionImplicit2022.md`
- created: `wiki/methods/ddim.md`
- created: `wiki/concepts/non-markovian-diffusion.md`
- updated: `wiki/concepts/score-sde.md` —— 接上 DDIM ODE 视角 = probability-flow ODE 离散化；补 sources 字段
- updated: `wiki/concepts/diffusion-process.md` —— 新增「Forward 不必是马尔可夫链」小节，链向 non-markovian-diffusion；补 sources
- updated: `wiki/methods/ddpm.md`、`wiki/sources/hoDenoisingDiffusionProbabilistic2020.md` —— 下游 DDIM 引用改 wikilink（已 ingest）
- updated: `wiki/overview.md` —— working thesis v0.1 → v0.2：推论 3「采样速度是开放赛道」被 DDIM 正向验证（加速 = 纯采样期、非马尔可夫族、inversion 入口），并强化推论 1 可变性光谱；「待调研方向」采样加速线刷新
- updated: `index.md` —— 刷新 updated（新页由 Dataview 自动收录）
- updated: `raw/literature-notes/songDenoisingDiffusionImplicit2022.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`

## [2026-05-14] thesis-update | overview 推论 1 重构：废弃"基础设施层/编辑层"两层划分
- updated: `wiki/overview.md` —— 推论 1 从"两层划分"改为"范式 + 可变性光谱"：唯一不变量是范式（迭代生成 + 预测噪声/速度场 + 沿生成链注入条件），组件按可变性排在光谱上；修正三处硬伤（U-Net 非固化、"一字不改"与 Flow Matching 矛盾、条件注入本身就要改 backbone）
- updated: `wiki/concepts/epsilon-parameterization.md`、`wiki/sources/hoDenoisingDiffusionProbabilistic2020.md` —— 同步：把旧"基础设施层"措辞改为"可变性光谱中'可演化但非主战场'一档"

## [2026-05-14] lint | wiki 全量 lint：修正 DDIM 年份矛盾、填充 overview 待调研方向、补 4 个 stub 页
- created: `wiki/methods/ncsn.md`、`wiki/methods/diffusion-2015.md`、`wiki/concepts/classifier-free-guidance.md`、`wiki/concepts/score-sde.md` —— Missing 项：被多页频繁提及但无独立页面的术语，建为 stub 待 ingest 原文扩充
- updated: `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md` —— C1：DDIM 年份 "Song 2021" → "Song et al., arXiv 2020 / ICLR 2021"；上下游引用改为 wikilink
- updated: `wiki/overview.md` —— S1：「待调研方向」从占位符填充为 6 条（score-based 上游 / Score SDE / CFG / DDIM 加速线 / 训练目标调参级修改 / 编辑方法族本体）；S2：`status` draft → active；NCSN / CFG 改 wikilink
- updated: `wiki/concepts/diffusion-process.md`、`wiki/concepts/score-matching.md`、`wiki/concepts/langevin-dynamics.md`、`wiki/methods/ddpm.md`、`wiki/entities/jonathan-ho.md` —— 反链：把 NCSN / Sohl-Dickstein 2015 / Score SDE / CFG 的纯文字引用改为指向新 stub 页的 wikilink
- updated: `index.md` —— 刷新 updated（新页由 Dataview 自动收录）
- updated: `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md` —— G2：删除无意义的空 `sources: []` 字段
- updated: `CLAUDE.md` §4 —— G2：新增 `source` 类型页例外说明（用 `raw:` 不用 `sources:`）
- 未处理（留给用户）：G1 空文件 `raw/notes/NoT Transformer in Noiser.md`（用户决定忽略）；G3 raw 文献笔记的空 wiki-links 占位段（用户手填）

## [2026-05-11] refactor | index.md 新增"📚 阅读清单"
- updated: `index.md` —— 在 Overview 之后插入 literature-notes 阅读看板，包含 4 个 Dataview 子块：🔥 优先阅读（P0/P1 未读完）/ 📖 在读 / ⭐ 高分已读 / 📋 全部 literature notes；附 status/priority/my-rating 字段速查

## [2026-05-11] refactor | CLAUDE.md 新增 §5.2 Refill 工作流
- updated: `CLAUDE.md` —— 新增 §5.2 `refill <citekey>` 命令：re-import 后快速回填 `ingested_to_wiki` / `wiki_page` / `created` / `updated`，无需走完整 ingest 流程；refill 不写 log.md

## [2026-05-11] refactor | literature-note template 全段 persist 化 + re-import frontmatter 回填规则
- updated: `templates/zotero/literature-note.md` —— 给 5 段用户可编辑正文加 `{% persist %}` 块（keys: `why-read` / `my-summary` / `wiki-links` / `thesis-implication` / `open-questions`），re-import 不再清空这些段落
- updated: `CLAUDE.md` §5.1 —— 新增"re-import 后的回填"规则：Claude 在 ingest（或被显式要求）时自动回填 `ingested_to_wiki` / `wiki_page` / `created`（`created` 从 wiki source 页读回真实首次 ingest 日期）；`status`/`priority`/`my-rating` 由用户自管
- updated: `raw/literature-notes/hoDenoisingDiffusionProbabilistic2020.md` —— 用户已完成迁移（5 段 persist 锚点注入，annotations 区已 append 两次 import），Claude 按新规则回填 `ingested_to_wiki: true`、`wiki_page`、`created: 2026-05-10`

## [2026-05-10] thesis-update | overview working thesis 推论 1 展开
- updated: `wiki/overview.md` —— 推论 1 展开为"两层栈 + 四个编辑层旋钮 + Flow Matching caveat"，明确 thesis 可行差异空间

## [2026-05-10] ingest | Denoising Diffusion Probabilistic Models (Ho et al. 2020)
- created: `wiki/sources/hoDenoisingDiffusionProbabilistic2020.md`
- created: `wiki/concepts/diffusion-process.md`
- created: `wiki/concepts/variational-bound-elbo.md`
- created: `wiki/concepts/epsilon-parameterization.md`
- created: `wiki/concepts/score-matching.md`
- created: `wiki/concepts/langevin-dynamics.md`
- created: `wiki/concepts/reparameterization-trick.md`
- created: `wiki/methods/ddpm.md`
- created: `wiki/entities/jonathan-ho.md`
- created: `wiki/entities/pieter-abbeel.md`
- created: `wiki/entities/uc-berkeley.md`
- created: `wiki/benchmarks/cifar10.md`
- created: `wiki/benchmarks/lsun.md`
- updated: `wiki/overview.md` —— working thesis 从空 → v0.1（全局+渐进、ε-pred 为基础设施层、编辑差异主要发生在 reverse 链注入方式）
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/hoDenoisingDiffusionProbabilistic2020.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`

## [2026-05-05] refactor | add zotero literature-note workflow
- created: `templates/zotero/literature-note.md`（从 `templates/literature-note.md` 移入）
- created: `raw/literature-notes/`
- updated: `CLAUDE.md` —— 目录契约新增 `raw/literature-notes/`，§5.1 增写 Zotero 文献笔记 ingest 规则与 frontmatter 回填例外

## [2026-05-05] init | vault scaffolded
- created: `CLAUDE.md`、`README.md`、`index.md`、`log.md`、`wiki/overview.md`
- created: `templates/{source,entity,concept,method}.md`
- created: `research/{thesis,ideas,experiments,related_work,outline}.md`
- created: `raw/{papers,articles,talks,notes,assets}/`、`wiki/{entities,concepts,methods,benchmarks,sources,comparisons,synthesis}/`
- 方法论参考：[[Karpathy's_Wiki_Method/llm-wiki]]
