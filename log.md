# Log

> Append-only 时间线。每条 entry 以 `## [YYYY-MM-DD] <op> | <subject>` 起始，便于 `grep "^## \[" log.md | tail`。

## [2026-07-30] ingest | shaulBespokeSolversGenerative2023 Bespoke Solvers for Generative Flow Models (Shaul et al. 2023)
- created: `wiki/sources/shaulBespokeSolversGenerative2023.md`
- updated: `raw/literature-notes/shaulBespokeSolversGenerative2023.md`（ingested_to_wiki: true）
- updated: `wiki/overview.md`（采样加速线加入 Bespoke Solvers）
- updated: `wiki/sources/chaTrainingFreeRefinementFlow2026.md`（奇异点统一框架段落同步修正：$l$ 新定义 + OOD 双条件 + 弱化断言）
- 关系：全局离线 solver 优化路线，与 FDS spatial / 我的 temporal adaptive 正交；可组合

## [2026-07-29] report | 组会 + 方向 pivot 落地 + solver 调研启动
- created: `reports/meetings/2026-07-29-group.md` —— 小组会简报（展示多模态问题 idea + 问科研流程）
- created: `reports/weekly/2026-W30.md` —— 首份周报 draft（7/21–7/28）
- updated: `reports/state.md` —— 正式化：FM 多模态 P0，energy 降 P2，bridge 关闭；组会后新增 solver 调研任务（T-20260729-01/02），toy 实验降 P1
- updated: `research/notes/2026-07-28-singularity-unified-framework.md` —— 修正 $l$ 定义（空间分辨率，非转向）；弱化断言为待验证假设；OOD 机制双条件
- 组会反馈：idea 有价值但可能重复，导师建议不急实验、先想清楚问题、看 solver 论文
- 新建待读列表：Optimal Stepsize / Instance-Aware Discretizations / SADA / Isokinetic FM / DPM-Solver 系 / SANA（师兄推荐）

## [2026-07-28] ingest | 2604.04646 Training-Free Refinement of Flow Matching with Divergence-based Sampling (Cha et al. 2026, ECCV'26)
- created: `wiki/sources/chaTrainingFreeRefinementFlow2026.md` —— FDS：用 velocity divergence 做 inference-time discrepancy proxy，零阶随机扰动选 divergence 最低候选做 spatial refinement；plug-and-play 叠加任何 solver；直接超过 HRF 等 training-based 方法
- updated: `raw/literature-notes/chaTrainingFreeRefinementFlow2026.md` —— 补全 why-read / my-summary / wiki-links / thesis-implication / open-questions
- updated: `wiki/overview.md` —— 采样加速线加入 FDS 引用 + thesis 相关性说明；sources 补入
- 判断：**高度相关**，与 thesis 方向一致（inference-time, training-free）；divergence + energy-guidance 组合是明确的下一步实验方向

## [2026-07-27] ingest | 2502.17436 Towards Hierarchical Rectified Flow (Zhang et al. 2025, ICLR'25)
- created: `wiki/sources/2502.17436-towards-hierarchical-rectified-flow.md` —— 在 velocity space 再跑一层 RF 学加速度，嵌套耦合 ODE 捕捉多模态速度分布；轨迹可交叉更直；低 NFE 有优势但参数量增大
- updated: `wiki/methods/rectified-flow.md` —— 新增「变体与扩展」节引用 HRF；sources 字段补入；open questions 补 HRF 相关
- 判断：对 text-guided editing 的平均抹消帮助有限（无条件生成验证、未做 editing 实验、仅 32×32）；不更新 working thesis

## [2026-07-24] lint | wiki 全面检查 + 整改
- fixed: `wiki/sources/albergoStochasticInterpolants2023.md` broken link `[[raw/articles/2303.08797v4]]` → `[[raw/articles/2303.08797v4.pdf]]`
- updated: `wiki/overview.md` — sources 字段加入 DiT；文本中 2 处 DiT 补 wikilink；updated 刷新
- updated: 8 页补 `[[wiki/sources/peeblesScalableDiffusionModels2023|DiT]]` 入链（flux, black-forest-labs, stable-diffusion, flux-kontext, controlnet, labsFLUX1KontextFlow2025, zhangAddingConditionalControl2023）
- batch: 41 页 status draft → stable（37 entities/benchmarks + 4 concepts/methods，均 30+ 天无更新）

## [2026-07-24] ingest | peeblesScalableDiffusionModels2023 (DiT)
- created: `wiki/sources/peeblesScalableDiffusionModels2023.md` —— Scalable Diffusion Models with Transformers
- 内容：用 ViT 替代 U-Net 做扩散骨干；adaLN-Zero 条件注入；scaling law（Gflops ↔ FID）；ImageNet 256 FID 2.27 SOTA
- 关联：[[wiki/methods/ldm]]、[[wiki/concepts/classifier-free-guidance]]、[[wiki/concepts/conditional-diffusion]]
- updated: `raw/literature-notes/peeblesScalableDiffusionModels2023.md` → ingested_to_wiki: true

## [2026-07-21] report | 科研汇报体系 schema v1 落地（per CLAUDE_CODE_REPORTING_SYSTEM_SPEC.md）
- created: `raw/worklogs/`（用户每日原始记录，LLM 只读）、`reports/{weekly,meetings,blockers}/`
- created: `reports/state.md` —— 唯一状态源骨架；初始化仅含「待用户确认迁移项」草案（拟 T/B/R/D 各 1–2 条，全部 needs-confirmation，未写入正式状态表）
- created: `reports/dashboard.md` —— 汇报入口 + Dataview（最近 8 周报 / active Blocker / 最近 8 会议简报）
- created: `templates/daily-worklog.md`、`templates/weekly-report.md`、`templates/blocker-report.md`、`templates/meeting-brief.md`
- updated: `CLAUDE.md` —— 架构表 3→4 层（+Reports）；目录契约 +`raw/worklogs/`、+`reports/`；新增 §9 Report 工作流（weekly/blocker/meeting/status/sync 五命令 + 状态词/结论强度/事实约束/ID 规则）；op 集合 +`report`（仅 confirmed 周报、确认的 Blocker 关闭、重大 schema 变更写日志，draft 不写）；Self-check +5 条报告类不变量；原 §9/§10 顺延为 §10/§11
- updated: `GUIDANCE.md` —— 四层结构说明 + 「科研汇报体系」用法节
- updated: `index.md` —— 新增 Reporting 入口 + 周报/Blocker/会议简报三个 Dataview 列表
- updated: `README.md` —— 工作记录节加「历史快照」说明（不重写既有记录）
- 边界遵守：`raw/` 既有文件、`research/` 全部内容、`wiki/` 均未改动；不回填历史周报；state 正式表待用户确认迁移项后再填

## [2026-06-29] ingest | FMPS: Flow Matching Posterior Sampling (Song et al. 2025)
- created: `wiki/sources/songFlowMatchingPosterior2025.md` —— source 页（速度↔score 桥 Prop1 + FreeDoM 式距离能量 + gradient/free 两种 $\hat x_0$ + $g^1$ 归一化；含用户 🔴"a,b 反了"待核）
- created: `wiki/methods/fmps.md` —— 方法页（family=guidance；FreeDoM 的 flow matching 版）
- updated: `wiki/methods/freedom.md`（加 flow 版 FMPS）、`wiki/concepts/conditional-diffusion.md`（clean-estimate 代表加 FMPS-flow）、`wiki/concepts/training-free-guidance.md`（flow 角度补 FMPS + 后验均值 caveat）
- updated: `wiki/synthesis/energy-guidance-landscape.md` —— ①行改☠️死(FMPS占)、②③改🔴占、§4 收束标作废
- updated: `research/ideas.md` —— energy-guidance 候选大改：FMPS 把①坐死 + 三段全红 + 收束作废 + 引用图硬证据；status → "公开文献无 carve，存活仅靠导师在研线"
- updated（raw 回填，§5.1）：`raw/literature-notes/songFlowMatchingPosterior2025.md` —— `ingested_to_wiki: true`、`wiki_page` 填好、补 wiki-links 节
- updated: `index.md` —— updated 刷新

## [2026-06-24] lint | wiki 整改：overview 同步 + 补 Missing 页 + 综述沉淀
- 扫描结果：broken link 0、orphan 0、frontmatter 缺失 0（结构层健康）
- updated: `wiki/overview.md` —— 推论 4 加"2026-06-24 方向复审"：bridge-SDE 暂挂、活线转 energy-guidance（与 research 层对齐）；sources +EGSDE/FreeDoM；updated 刷新
- created: `wiki/concepts/tweedie-formula.md` —— Missing（6 页提及无页）：后验均值 $\hat x_0$ 公式 + 推导 + flow 变体
- created: `wiki/methods/dps.md` —— Missing（6 页提及）：DPS draft 骨架（待 ingest 原文），FreeDoM 最近邻
- created: `wiki/synthesis/energy-guidance-landscape.md` —— 沉淀 sweep 四落点 + 师兄三段框架 + 结构化 E + 收束 flow（顺带给 afhq 补第二入链）
- updated: `wiki/concepts/training-free-guidance.md` —— 加 TFG 别名（5 页提及，并入此页不另开）
- updated: `index.md` —— updated 刷新
- 未做（判断为低价值）：Dhariwal/UGD/MPGD/ILVR/CycleGAN 独立页——已在相关页标"待 ingest"，提及即可

## [2026-06-24] thesis-update | energy-guidance 候选升级：sliver 已定（师兄三段框架）
- updated: `research/ideas.md` —— 顶部候选条从"待 sweep"升到"sliver 已定（待第一个实验）"：补 sweep 结论（generic 全红、因导师在研线不 KILL）+ 师兄 6/23 三段框架（①$\hat x_0$ 估计 ②energy 获取 ③→guidance，逐段 heuristic→原理化）+ 核心收束"三段原理化方向全指向 flow/RF 底座"+ 下一步（三段梳理 + 第一个最小实验设计）
- 触发：与师兄 6/23 对话确认 FreeDoM=baseline、三段改进框架；本轮在 chat 完成第③段（energy→guidance: MPGD/TFG/步长/流形）盘点

## [2026-06-23] ingest | FreeDoM: Training-Free Energy-Guided Conditional Diffusion (Yu et al. ICCV 2023)
- created: `wiki/sources/yuFreeDoMTrainingFreeEnergyGuided2023b.md` —— source 页（clean-estimate 路线：Tweedie $\hat x_0$ + 现成距离能量 + time-travel；含用户 🔴"eq4 Z 没算"批注的交叉引用）
- created: `wiki/methods/freedom.md` —— 方法页（family=guidance；pipeline 伪代码；三大局限）
- created: `wiki/concepts/training-free-guidance.md` —— 新概念（免训练引导 + TFG 统一设计空间表：DPS/FreeDoM/UGD/LGD/MPGD）
- updated: `wiki/concepts/energy-guidance.md` —— clean-estimate 行 FreeDoM 加链；出处补 FreeDoM + 母页/子族
- updated: `wiki/methods/egsde.md` —— 加 clean-estimate 对照（FreeDoM）
- updated: `wiki/sources/zhaoEGSDEUnpairedImagetoImage2022.md` —— open-question FreeDoM 标 ✅ 已 ingest
- updated: `wiki/concepts/classifier-guidance.md` —— 加"免训练版 = training-free-guidance/FreeDoM"
- updated（raw 回填，§5.1 四字段）：`raw/literature-notes/yuFreeDoMTrainingFreeEnergyGuided2023b.md` —— `ingested_to_wiki: true`、`wiki_page` 填好
- updated（raw 正文，用户授权"错的就修"）：FreeDoM 笔记 why-read / my-summary / wiki-links 的 misnamed + 缺页 wikilink 已修正（DPS/reward-guidance 等无页项转纯文本）
- 未动：`status` 用户自定；overview working thesis（energy-guidance 仍红海，待师兄定 sliver）

## [2026-06-23] ingest | conditional-diffusion（用户手写推导 raw/notes）
- created: `wiki/concepts/conditional-diffusion.md` —— 概念母页（贝叶斯拆条件 score → Tweedie → 点估计 → 能量梯度；含归一化 $Z$ 两种 framing + 用户 cat-dog 洞见 + noisy-aligned vs clean-estimate 对照表）
- 来源：`raw/notes/conditional diffision.md`（用户手写，已自行改对符号 $s_0-\nabla_y E$ 并补 $Z'$ 推导）；按 §1，raw/notes 非 literature-notes，**正文未改一字**
- 关系：成为 classifier-guidance / energy-guidance / training-free-guidance / EGSDE / FreeDoM 的数学母页

## [2026-06-21] ingest | EGSDE: Unpaired I2I via Energy-Guided SDE (Zhao et al. NeurIPS 2022)
- created: `wiki/sources/zhaoEGSDEUnpairedImagetoImage2022.md` —— source 摘要页（能量引导反向 SDE + 双专家 + PoE + noisy-aligned 设计 + thesis 接口）
- created: `wiki/methods/egsde.md` —— 方法页（family=guidance；冻结生成器 + 采样期能量梯度；pipeline 伪代码）
- created: `wiki/concepts/energy-guidance.md` —— 新概念（classifier guidance 的能量化推广 + PoE + noisy-aligned vs clean-estimate 设计轴）
- created: `wiki/benchmarks/afhq.md` —— AFHQ 数据集页（Cat→Dog / Wild→Dog；unpaired I2I realism↔faithfulness 双量）
- updated: `wiki/concepts/classifier-guidance.md` —— 加"推广版 = energy-guidance/EGSDE"链
- updated: `wiki/methods/sdedit.md` —— 下游加 EGSDE（SDEdit 是其 p_r1 realism 专家）
- updated: `wiki/entities/jun-zhu.md` —— 关键工作加 EGSDE；合作者加 Chongxuan Li / Min Zhao；sources +EGSDE
- updated: `index.md` —— updated 时间戳（Dataview 自动收录新页）
- updated（raw 回填，§5.1 例外 + 用户本次显式授权改 raw）：`raw/literature-notes/zhaoEGSDEUnpairedImagetoImage2022.md` —— `ingested_to_wiki: true`、`wiki_page` 填好；修 my-summary #4 公式缺等号 + `# $$` 标题 bug；修 why-read 错名 wikilink；补 wiki-links 节
- 未动：raw 的 `status/priority/my-rating`（用户自定）；`wiki/overview.md` working thesis 版本号（thesis 仍 🔁 复审中，energy-guidance 升级待 sweep）

## [2026-06-18] refactor | 削减 FlowCycle 占比 + thesis 方向标复审 + energy-guidance 入候选
- updated: `wiki/overview.md` —— thesis 押注的 ODE 侧成熟论据去 FlowCycle，RF-Inversion 接管
- updated: `wiki/methods/ddbm.md`、`wiki/methods/rectified-flow.md` —— open-question 去 FlowCycle 接合话术，保留技术点
- updated: `wiki/sources/liuFlowStraightFast2022a.md`（3 处）、`wiki/sources/zhouDenoisingDiffusionBridge2023.md`、`wiki/sources/labsFLUX1KontextFlow2025.md` —— 去 FlowCycle 命名，smoother/coupling 技术点保留
- updated: `README.md` —— 6.1-6.7 止损结论去 FlowCycle 弱点分析
- updated: `research/thesis.md` —— 去 FlowCycle（RF-Inversion 接管）；v0.1 标 🔁 方向复审中（energy-guidance 候选待 sweep）
- updated: `research/ideas.md` —— Active 顶部新增 [2026-06-18] energy-guided conditional generation 候选（EGSDE 路线，待 3-sweep）；删"FlowCycle 弱点分析"下一步项；KILL 标题去 FlowCycle-SDE
- 未动（按契约）：`raw/literature-notes/liuFlowStraightFast2022a.md` line 97/223 用户手写批注仍含 FlowCycle，§1 raw 只读，留用户自删；log 历史条目不改

## [2026-06-02] ingest | FLUX.1 Kontext: Flow Matching for In-Context Image Gen & Editing (Black Forest Labs 2025)
- created: `wiki/sources/labsFLUX1KontextFlow2025.md` —— source 摘要页（条件生成 $p(x|y,c)$ + 潜空间 RF + in-context token 拼接 + KontextBench 结果，PDF 交叉验证）
- created: `wiki/methods/flux-kontext.md` —— 方法主页（family=editing；与 SDEdit/ControlNet/attention-injection 条件注入分界表）
- created: `wiki/concepts/in-context-conditioning.md` —— **第六条条件注入通道**（上下文图作 latent token 序列拼接 + 3D RoPE）；区别于 noising/cross-attn/attention-injection/sideband
- created: `wiki/benchmarks/kontextbench.md` —— vault 首个 in-context 编辑 benchmark（1026 pairs / 5 任务 / 人评 + AuraFace 角色相似度）
- created: `wiki/entities/black-forest-labs.md` / `flux.md`（具名模型族）
- updated: `wiki/overview.md` —— 主要派系「flow-matching-based」首篇 ✅ FLUX.1 Kontext；新增第六条注入通道 in-context-conditioning 注（编辑主线 = 条件 FM，非 bridge）
- updated: `wiki/methods/rectified-flow.md` —— 下游/工业落点/open-question 标 FLUX.1 Kontext ✅；sources +Kontext
- updated: `wiki/concepts/flow-matching.md`、`wiki/entities/robin-rombach.md`（+BFL/FLUX/Kontext，sources +Kontext）
- updated: `index.md`；`raw/literature-notes/labsFLUX1KontextFlow2025.md` 回填 ingested_to_wiki/wiki_page（§5.1；status/priority/my-rating 未动）
- 关键 takeaway（对 thesis）：BFL 工业旗舰把统一编辑做成 **in-context 条件 FM**，**不走 bridge**——再次印证 bridge-editing 是支线、编辑主线在 in-context 条件生成；Kontext 是绕不开的强 baseline + KontextBench 是绕不开的评测，故"纯方法新意"更难、execution/窄 niche 更现实（Kontext 弱项：global editing / 风格参考）
- thesis：working thesis / research/thesis 未改（用户 thesis-implication 原文留空）

## [2026-06-01] query | diversity-editing sweep + 止损 KILL bridge-editing 全家桶
- 触发：用户"sweep"验证"可控多样性 / 一对多 text editing"carve 是否开放
- 结果：🔴 **也被占** —— [OSCAR (2510.09060)](https://arxiv.org/abs/2510.09060)（training-free 正交随机控制提多样性、不损 fidelity）、Variational Rectified Flow Matching（latent 捕多模）、Discretized-RF、FlowSlider（fidelity-steering 旋钮）；one-to-many 编辑自 Blended Diffusion 2022 即有
- updated: `research/ideas.md` —— **重构**：Active 清空为"无存活方法 carve + 下一步(FlowCycle 弱点/窄 niche/导师 brief)"；Killed 新增"Bridge-SDE editing 全家桶"条（逐落点撞车证据：加速=CDBM、inversion=DBIM、ODE 编辑=RF-Inversion 系、FlowCycle-SDE=derivative、diversity=OSCAR、理论=SI/SB）
- updated: `wiki/synthesis/bridge-sde-editing-landscape.md` —— text-editing 格标 🔴 也被占；"对 thesis 指向"改为止损实况（三次 sweep 三次撞 → 红海，方向交回导师）
- meta 结论：armchair 想方法→sweep→撞，循环三次即证明找题方法错；红海出论文靠 execution+窄 niche+导师在研线
- thesis：[[research/thesis]] v0.1 不动

## [2026-06-01] ingest | Stochastic Interpolants (Albergo, Boffi & Vanden-Eijnden 2023, JMLR) — cold ingest from PDF
- 触发：用户验证"让 SDE bridge 脱离 diffusion 框架"是否撞车（无 Zotero 文献笔记，直接读 raw/articles/2303.08797v4.pdf，80 页）
- created: `wiki/sources/albergoStochasticInterpolants2023.md` —— SI source 摘要页（核心构造 + ODE/SDE 解耦 + likelihood control + Schrödinger Bridge 还原 + Fig 15 多样性机制）
- upgraded: `wiki/concepts/stochastic-interpolants.md` —— draft → **active**，真实内容重写（含 SI vs DDBM/DBIM/FM 分工表）；sources 指向新 source 页
- created: `wiki/entities/michael-albergo.md` / `eric-vanden-eijnden.md` / `nyu-courant.md`
- updated（关链 ✅）：`wiki/concepts/flow-matching.md`、`wiki/methods/rectified-flow.md`、`wiki/sources/liuFlowStraightFast2022a.md` —— "Stochastic Interpolants 待 ingest" → ✅ 已 ingest + 链 source
- updated: `wiki/synthesis/bridge-sde-editing-landscape.md` —— 理论侧"统一与构造"标 ✅ ingest；优先阅读清单①标 ✅；点明 SI Conclusion 自指 inpainting/SR 应用口
- updated: `wiki/overview.md` —— Bridge 待调研行：SI/DBIM 标 ✅ ingest，新增"纯 bridge-SDE 理论/脱离 diffusion 已验明 KILL（撞 SI）"；frontmatter +SI source
- updated: `research/ideas.md` —— Active 加 SI 应用口；**Killed 新增**"脱离 diffusion 框架"条（逐条撞车证据 + 直觉正确但转应用）
- **关键判决（对用户选题）**：用户"脱离 diffusion 的 SDE bridge"理论想法 = SI 本身（自由插值/任意两分布/ODE+SDE 可调噪声/还原 SB），理论侧已满 → KILL；但 SI 自指 inpainting/SR/forecasting 应用口 + Fig 15 可调 ε 多样性机制，把出路指回"editing 应用 + diversity 旋钮"
- 注：raw/articles/ 非 literature-note，无 §5.1 frontmatter 回填

## [2026-06-01] ingest | Diffusion Bridge Implicit Models / DBIM (Zheng, He, Chen, Bao & Zhu 2025, ICLR)
- created: `wiki/sources/zhengDiffusionBridgeImplicit2025.md` —— DBIM source 摘要页（Table 2 translation + ImageNet inpainting + η 消融真实数字，从 Zotero PDF 取）
- created: `wiki/methods/dbim.md` —— 方法主页（family=bridge）：DDBM 的 DDIM 化，training-free 25× 加速 + 确定可逆桥
- created: `wiki/entities/kaiwen-zheng.md` / `jun-zhu.md` / `tsinghua-university.md` —— 一作 / 资深作者 / 机构（THU-ML）
- updated: `wiki/concepts/non-markovian-diffusion.md` —— **新增 Bridge 版小节**：DBIM:DDBM = DDIM:DDPM 对位表 + booting noise；sources +DBIM
- updated: `wiki/concepts/probability-flow-ode.md` —— 加"桥的隐式 ODE（DBIM）解决 DDBM 纯 ODE 奇异 + 双向确定→encoding/reconstruction"；sources +DBIM
- updated: `wiki/concepts/diffusion-bridge.md` —— 加 DBIM 快速采样/确定可逆条；sources +DBIM
- updated: `wiki/methods/ddbm.md` —— 关系网加 DBIM/CDBM 快速采样行；待补项标注 inversion 原语已由 DBIM 提供
- updated: `wiki/synthesis/bridge-sde-editing-landscape.md` —— 网格拆出 inversion（DBIM 已占）vs text-editing（仍开）两行；新增 DBIM 定位节；选题再收窄为"DBIM 可逆桥之上的 target-aware cycle-consistent text-editing"；I³SB 降级（被 DBIM 覆盖）；sources +DBIM
- updated: `research/ideas.md` —— bridge 条加 2026-06-01 DBIM ingest 后的再收窄
- updated: `raw/literature-notes/zhengDiffusionBridgeImplicit2025.md` —— 回填 `ingested_to_wiki: true` / `wiki_page`（§5.1 例外）；status/priority/my-rating 未动（用户自管）
- updated: `index.md` —— 刷新 updated
- thesis：按惯例 **不改** overview/thesis.md，source 页 thesis-implication 留空（列候选）；关键 implication = DBIM 提供 bridge inversion 原语，把 bridge-SDE 编辑选题从"做 inversion"再收窄为"在可逆桥上做 cycle-consistent text-editing"
- 关键 takeaway：DBIM = 非马尔可夫桥族保边缘 → 复用 DDBM score 不重训；ρ 控随机（ρ=0 确定 ODE+booting noise→可逆）；25× 加速；η 消融给出"确定性利可逆/收敛、随机性利多样性"的实证

## [2026-06-01] lint | DDBM ingest 后整改：消 1 orphan + raw priority 规范化
- 扫描：81 wiki 页 · 全部 wikilink + .webp 嵌入解析 · **broken 0**（50+ grep 报告均误报：markdown 表格 `\|` 转义 + raw/assets .webp 嵌入 + research/CLAUDE 脚手架占位符）/ **frontmatter 缺字段 0** / **矛盾 0**
- updated: `wiki/overview.md` —— Orphan 修复：待调研方向 Bridge 行加 [[wiki/synthesis/bridge-sde-editing-landscape]] 链
- updated: `wiki/methods/ddbm.md` —— 关系网加 landscape 反链
- updated: `research/ideas.md` —— 2026-06-01 条加 landscape 全景链（landscape 入链 0 → 3，orphan 消除）
- updated: `raw/literature-notes/zhouDenoisingDiffusionBridge2023.md` —— Suggestion 采纳（用户授权）：`priority: p5 → P1`，使其进 index「🔥优先阅读」看板（§5.1 用户自管字段，经确认才改）
- Missing 候选（识别但本次不建，按 thesis 距离）：**P1** EDM(Karras 2022，4 页)/ I²SB(Liu 2023，5 页，bridge-SDE 选题最近 baseline)；**P2** Schrödinger Bridge 概念页(5 页)；**P3** DDIB/BBDM(各 2 页)
- 复跑校验：broken 0 / orphan 0 / weak 0
- 触发：用户"sweep 一下"——确认 [[research/ideas]] bridge-SDE 选题的 prior-art 边界
- created: `wiki/synthesis/bridge-sde-editing-landscape.md` —— bridge SDE/ODE × 任务网格 + 关键论文定位（均未 ingest，仅 arXiv 链接）；后补「理论侧 landscape」节 + 两摞优先阅读清单
- 理论侧 sweep 结论：bridge-SDE 理论已基本合拢（统一=stochastic interpolants 2303.08797；lens 已成文=2509.24531；SB 收敛/最优 stochasticity/误差界 2024–2026 正被专业概率组钉死）——**不宜作主选题**；若要理论味则做应用毗邻理论（随机桥上 inversion/cycle-consistency 闭合条件）
- updated: `research/ideas.md` —— 2026-06-01 条加 sweep 结果：**#2 加速 KILL**（CDBM NeurIPS'24 / Inverse Bridge Matching Distillation）；FlowCycle 身份确认 = HKUST Long 组 inversion-FREE flow editing（修正我此前"RF-inversion 往返"的误解）；选题收窄为"随机桥上的 target-aware cycle-consistent editing（FlowCycle 的 SDE 推广）"；列待确认 4 篇
- 关键发现：bridge-ODE 侧 inversion/editing 红海（RF-Inversion/RF-Solver-Edit/OT-for-RF）、SDE 侧加速已满（CDBM）；相对开放格 = SDE 侧 cycle-consistent text-editing
- 未改 thesis（用户决定：待 4 篇精读后再定 v0.2）

## [2026-06-01] thesis-update | overview v0.4 → v0.5（立 bridge-SDE 为方向）+ research/ 首次实质写入
- 用户决定（选项 B）：把 bridge-SDE 立为 thesis 方向；并把候选 idea 写入 research/（带 prior-art 边界）
- updated: `wiki/overview.md` —— 标题 v0.4 → v0.5（+ DDBM/Bridge）；**新增推论 4**：transport 为比 denoising 更高一层的候选不变量、SDE/ODE realization 为正交结构轴、bridge-SDE 侧欠发达 = 施力点；附三条诚实 caveat（lens 非原创=stochastic interpolants / DDBM 是 translation 非 editing / 非 greenfield）；重审注从"三条推论"改为含推论 4 待验证；待调研方向新增 Bridge 线；frontmatter +DDBM source
- created（research 区，§8 经用户确认）：
  - `research/thesis.md` —— **首个实质版本 v0.1 草稿**（标注待用户精修）：research 问题 = bridge-SDE inversion/editing；核心论点 3 条 + 诚实边界 3 条；全部 wikilink 引 wiki，无复制
  - `research/ideas.md` —— Active 新增 [2026-06-01]「Bridge-SDE inversion/editing（接 FlowCycle）」：用户原创 framing + 子问题 #1–#4 + prior-art 边界（stochastic interpolants / I²SB / SB / BBDM；#2 加速须查 2024）
- 校准记录：明确告知用户"bridge SDE vs bridge ODE"lens ≈ stochastic interpolants（Albergo），非原创；可 defensible 的卖点是 bridge-SDE 侧欠发达，最近选题 = bridge-SDE inversion（复用 FlowCycle）
- 注：thesis.md / ideas.md 为草稿，最终措辞与押注力度由用户校准

## [2026-06-01] ingest | Denoising Diffusion Bridge Models / DDBM (Zhou, Lou, Khanna & Ermon 2023)
- created: `wiki/sources/zhouDenoisingDiffusionBridge2023.md` —— DDBM source 摘要页（含 Fig 1 forward bridge SDE / reverse PF-ODE 嵌入；Table 2 translation + Table 4 unconditional 真实数字，从 Zotero PDF 交叉验证取得）
- created: `wiki/methods/ddbm.md` —— 方法主页（family=**bridge**，vault 首个 bridge family；与 RF/SDEdit 对照表）
- created: `wiki/concepts/infinitesimal-generator.md` —— 生成元 → Kolmogorov backward → Fokker-Planck（伴随）→ h-transform 生成元；骨架来自用户手写 [[raw/notes/生成元方法对于SDE]]
- created: `wiki/concepts/doob-h-transform.md` —— 终点吸引漂移 $g^2\nabla\log h$ 的来源；DDBM 公式 5 的机制
- created: `wiki/concepts/diffusion-bridge.md` —— 双端点随机桥；bridge SDE vs bridge ODE 对照表；SB/Doob/Brownian 谱系
- created: `wiki/concepts/stochastic-interpolants.md` —— **draft 枢纽页**（Albergo 原文待 ingest）：bridge SDE vs bridge ODE 的严格统一归宿
- created: `wiki/entities/linqi-zhou.md` / `aaron-lou.md` —— 一作 / 二作 stub
- updated: `wiki/concepts/score-sde.md` —— 关系加"DDBM = Score SDE 经 Doob h 推广到 paired 端点；源端高斯严格退化"；sources +DDBM
- updated: `wiki/concepts/probability-flow-ode.md` —— 加"桥版 PF-ODE（公式 7）+ 纯 ODE 采样糊、需 stochasticity"；sources +DDBM
- updated: `wiki/concepts/flow-matching.md` —— Albergo 裸文本 → 链 stochastic-interpolants；加"bridge ODE vs bridge SDE + DDBM 统一是有条件约化非特例"；sources +DDBM
- updated: `wiki/concepts/fokker-planck-equation.md` —— 关系加"FPE = 生成元伴随、backward 作用在观测函数"链 infinitesimal-generator；sources +DDBM
- updated: `wiki/methods/rectified-flow.md` —— Stochastic Interpolants 裸文本 → 链新页；加 DDBM bridge SDE 对位 + RF 跨域崩盘（DIODE 77.18）实证；sources +DDBM
- updated: `wiki/entities/stefano-ermon.md` / `stanford.md` —— 回填 DDBM 工作 + Linqi Zhou / Aaron Lou 链接
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/zhouDenoisingDiffusionBridge2023.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（§5.1 例外）；**未动** `status`(unread) / `priority`(p5) / `my-rating`(1)，留用户重设
- thesis：按用户指示 **不动** working thesis、source 页「对我的 thesis 的启示」留空（仅列候选角度待用户取舍）；overview 未改
- 讨论结论（ingest 前对齐）：DDBM 对 OT-FM 的"unification"是话术——仅 noiseless 极限 $c\to0$+VE 的有条件约化（§6.1 Case 2），严格退化只对 diffusion（Case 1）；bridge SDE（DDBM）vs bridge ODE（flow）的真正公约数是 stochastic interpolants（论文自承用不同 loss）
- 待 ingest（按 thesis 距离）：Stochastic Interpolants（Albergo，统一框架）；I²SB / Schrödinger Bridge；EDM（Karras 2022，DDBM 的 preconditioning+sampler 基础）；DDBM 上的 editing/inversion 后续（接 FlowCycle）

## [2026-05-29] query | Score ∇log p vs Velocity Field v（保守场 vs 一般场）
- 触发：用户 query「SDE 和 FM 的 ODE 是并列的吗」+ 追问"语义上 score 能写成 ∇log p、FM 的 v 不能"
- created: `wiki/comparisons/score-vs-velocity-field.md` —— 归档该对比：结果层（仅高斯路径可线性互转）vs 语义层（score 必保守场、v 一般非保守，写不成 ∇(·)）；diffusion PF-ODE 速度场恒保守（VE/VP 验证）；FM 把动力学从保守场子集解放到一般场 = OT 能走直线的根源
- updated: `wiki/concepts/flow-matching.md` —— 「与 score-based 分工」表后加"对象本质"指引；updated 2026-05-29
- updated: `wiki/concepts/probability-flow-ode.md` —— 关系节加"PF-ODE 速度场保守 vs FM 非保守"条；updated 2026-05-29
- updated: `wiki/overview.md` —— 推论 1「训练目标」一档加数学本质注（保守→一般场的解放 = RF 能拉直的根源）
- 不写 thesis 改动：working thesis v0.4 不变；本条是概念澄清沉淀，非新源 ingest

## [2026-05-29] ingest | SDEdit: Guided Image Synthesis and Editing with SDEs (Meng et al. 2022)
- created: `wiki/sources/mengSDEditGuidedImage2022.md` —— SDEdit source 摘要页（含原文 Fig 1 perturb 直观 + Algorithm 1 VE-SDE 伪代码嵌入）
- created: `wiki/methods/sdedit.md` —— 方法主页（family=**editing**，vault 首个 editing family 页）
- created: `wiki/concepts/noising-strength.md` —— $t_0$ / SD img2img `strength` 旋钮；realism↔faithfulness 单调曲线；推论 2 的核心量
- created: `wiki/entities/chenlin-meng.md`（一作，也是 DDIM 二作）/ `jiajun-wu.md` / `jun-yan-zhu.md`（CycleGAN/pix2pix 作者，GAN 编辑时代代表）/ `yutong-he.md`
- updated: `wiki/overview.md` —— **不升 working thesis 版本号**：(1) 推论 2 主体由"待编辑论文验证"→ ✅ **SDEdit 首次直接实证**（新增「首个直接编辑实证」块，含 $t_0$ vs CFG-$w$ 旋钮区分、单次 vs 全程介入对照）；(2) 主要派系第 1 类 **Inversion-based 改名 Inversion / noising-based**（用户选 a1），SDEdit ✅ 填入并排出"SDEdit→DDIM-inversion→Null-text"成本递增光谱；(3) 派系总前提"全部建立在 SD 上"修正为"多数"——SDEdit 是底座无关反例（早于 SD、套任意 score model）；(4) 重审注 + 待调研方向吸收 SDEdit；sources 加 [[wiki/sources/mengSDEditGuidedImage2022]]；updated 2026-05-29
- updated: `wiki/methods/ddim.md` —— 关联节加 SD img2img = 确定性 SDEdit；厘清 DDIM-inversion（带优化反演）vs SDEdit/img2img（直接加噪）同派系两端；updated 2026-05-29
- updated: `wiki/concepts/sideband-conditioning.md` —— inversion 一支补 SDEdit 作为"零成本下界"（连优化都没有），sideband 退化为"用 guide 替换初始噪声"；成本光谱细化为 SDEdit→DDIM-inv→Null-text→ControlNet；updated 2026-05-29
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/mengSDEditGuidedImage2022.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（§5.1 例外）
- thesis：working thesis v0.4 不变；但 SDEdit 是 **vault 首篇真正的 text/stroke-guided editing 论文**、且给推论 2 第一个直接实证——下次 ingest attention-injection 派系（Prompt-to-Prompt）后，推论 2 若再获一致支持，可考虑升 v0.5
- 用户决定：(a1) Inversion-based 派系改名 Inversion / noising-based 并纳入 SDEdit；(b) 建 noising-strength 概念页；(c) "SDEdit $t_0$ 是推论 2 首个直接实证" 按 🟣 级 thesis-implication 处理
- 待 ingest（按 thesis 距离）：Prompt-to-Prompt（attention-injection 首篇）；Null-text inversion / DDIM-inversion（noising-based 派系内更精细者，验证成本光谱）；SD3 / FLUX（flow-matching-based 首篇）；LoRA / T2I-Adapter（sideband-conditioning 上游）

## [2026-05-28] lint | ControlNet ingest 后整改：5 处 Stale 关闭 + SUG-1 措辞统一
- 扫描：64 wiki 页 · 全部 wikilink + raw/assets/ 图片嵌入解析 · broken **0**（1 false positive：[[research/thesis]] 在 Obsidian 正常解析，脚本未扫 research/）/ orphan **0** / weakly-linked **0** / frontmatter 缺失 **0** / 矛盾 **0**
- updated: `wiki/sources/rombachHighResolutionImageSynthesis2022.md`
  - S1（line 109）"下游编辑论文 ... ControlNet ..." 裸文本 → ControlNet 链 [[wiki/methods/controlnet]] 并标 ✅ 已 ingest；改写整段说"sideband 加性注入"
  - S2（line 133）open question P2 "待 ingest ControlNet 原文厘清" → ✅ 关闭，给出实际答案：ControlNet **未保留** LDM 的 concat 设计、改成 "复制 SD encoder + middle + zero conv 加性注入"，并对照差异（concat 仍要重训 vs sideband 冻主干）
- updated: `wiki/methods/ldm.md`
  - S3（line 15）family 注释 "ControlNet、Prompt2Prompt" 裸文本 → ControlNet 链 [[wiki/methods/controlnet]]；SD1.x/2.x 链 [[wiki/entities/stable-diffusion]]
  - S4（line 92）"几乎所有 inversion-based / attention-injection / ControlNet / instruction-tuned 方法"分类不齐 → 按 overview 新派系**重述为五类**：inversion-based / attention-injection / **control/sideband-injection** / instruction-tuned + flow-matching-based 注释
  - **SUG-1**：line 92 的措辞统一已并入 S4 修改
- updated: `wiki/concepts/latent-space-generative-modeling.md`
  - S5（line 65）"ControlNet feature 注入" → ControlNet 链 [[wiki/methods/controlnet|ControlNet]] sideband 注入
  - `updated` 刷 2026-05-27 → 2026-05-28
- 实际涉及 **3 个文件**（5 处 + SUG-1 中 S1+S2 同文件、S3+S4+SUG-1 同文件、S5 独立），非预估的 6 页
- Missing 候选（按 thesis 距离 + 提及频率）：
  - **P0**：Prompt-to-Prompt（8）/ Null-text Inversion（9）/ InstructPix2Pix（9）/ SD3（44）/ FLUX（42）
  - **P1**：T2I-Adapter（12）/ IP-Adapter（10）/ LoRA（14，sideband-conditioning 上游）/ VQGAN（12，perceptual-compression 上游）
  - **P2**：MasaCtrl / Plug-and-Play / StyleAligned / Classifier guidance 原文 / CFG 原文
  - **Schema-defer**：DDIM inversion 概念页（待 Null-text Inversion ingest 时一并建）
- Suggestion（建议但本次不动）：
  - **SUG-2**：raw/assets/zhangAddingConditionalControl2023-1779969409375.webp 是 size-identical 重复副本，未被任何文档 embed —— 用户手动删除（§1 raw 区只读）
  - **SUG-3**：sideband-conditioning 当前列 5 个实例（LoRA / Adapter / ControlNet / T2I-Adapter / IP-Adapter）但只有 ControlNet 有 wiki 页 —— **下次 ingest 优先 LoRA 或 T2I-Adapter** 而非 SD3 的论据（虽然 SD3 提及次数更高，但 LoRA / T2I-Adapter 直接提升 sideband 抽象的可证伪性）
  - **SUG-4**：overview 第 5 类派系成员中 T2I-Adapter / IP-Adapter / GLIGEN 仍是裸文本，待这些 ingest 后再补链
- 仍开放（用户决策）：下一轮 ingest 候选——Prompt-to-Prompt（验证 sideband 抽象覆盖 attention-injection）/ LoRA（sideband 上游）/ SD3（flow-matching-based 派系首篇）三选一

## [2026-05-28] ingest | Adding Conditional Control to Text-to-Image Diffusion Models / ControlNet (Zhang, Rao & Agrawala 2023)
- created: `wiki/sources/zhangAddingConditionalControl2023.md` —— ControlNet source 摘要页（attention/feature-injection 阵营之外的第一篇 sideband 注入代表）
- created: `wiki/methods/controlnet.md` —— 方法主页（family=other；附着 SD1.5）
- created: `wiki/concepts/zero-convolution.md` —— $1\times 1$ conv 零初始化，ControlNet 稳定 fine-tune 关键；与 LoRA $A=0$ / Adapter 末端零初始化思想同源、对照表
- created: `wiki/concepts/sideband-conditioning.md` —— 统一抽象「frozen backbone + trainable sideband + 初始化恒等」，覆盖 LoRA / Adapter / ControlNet / T2I-Adapter / IP-Adapter；明确为 thesis 在资源约束下的核心可行抽象
- created: `wiki/entities/stable-diffusion.md` —— **具名模型**条目（与方法页 [[wiki/methods/ldm]] 分开；版本谱系 SD1.x → SDXL → SD3）
- created: `wiki/entities/lvmin-zhang.md` / `anyi-rao.md` / `maneesh-agrawala.md`
- updated: `wiki/overview.md` —— **不升 working thesis 版本号，本次有结构性更新**：(1) 推论 1 关键修正(iii) 措辞由"动 backbone 内部"改为"触及 backbone 内部"，并加 ControlNet 作为"克隆结构作 sideband"的精确变奏注；(2) 推论 1 支持光谱新增 ControlNet 作为"sideband 注入"档（与 LDM "压缩层"对称——一前一侧）；(3) 推论 2 新增「全 $t$ sideband vs 逐 $t$ guidance」对照 caveat（thesis 必须回答的核心问题）；(4) 「主要派系」改为五类——新增 **Control/sideband-injection** (✅ ControlNet 已 ingest)，并以 sideband-conditioning 作为前四类的统一上层抽象；(5) 「重审注」吸收 ControlNet；sources 加 [[wiki/sources/zhangAddingConditionalControl2023]]；updated 2026-05-28
- updated: `wiki/methods/ldm.md` —— 下游编辑方法节标注 ControlNet ✅；具名落地权重指 [[wiki/entities/stable-diffusion]]
- updated: `wiki/concepts/cross-attention.md` —— 新增 "vs Sideband 注入" 一节对照（U-Net 内部 token K/V 化 vs U-Net 外部 sideband 加性 residual）
- updated: `index.md` —— 刷新 updated
- updated: `raw/literature-notes/zhangAddingConditionalControl2023.md` —— 回填 `ingested_to_wiki: true`、`wiki_page`（§5.1 例外）
- thesis：working thesis v0.4 不变（ControlNet 没动 paradigm、没动训练目标，是"组件维度→sideband 注入"上的新增；但本次 ingest 给出了 thesis 的**核心可行抽象**——frozen backbone + trainable sideband——值得考虑在下次 ingest（推荐 Prompt-to-Prompt 看 attention-injection 是否也能干净塞进该抽象）后升 v0.5）
- 用户决定：(a) "frozen backbone + sideband 是 thesis 在资源约束下的核心可行抽象" 按 🟣 级 thesis-implication 处理；(b1) 主要派系新增第 5 类 Control/sideband-injection（而非塞进 attention/feature-injection）；(c) 加 Stable Diffusion 具名模型 entity 页
- 待 ingest（按 thesis 距离）：Prompt-to-Prompt（attention-injection 派系首篇；最能验证 sideband 抽象是否覆盖该派系）；T2I-Adapter / IP-Adapter（Control/sideband 派系并行）；SD3 / FLUX（flow-matching-based 派系首篇）；LoRA / Adapter 原文（sideband-conditioning 概念页的上游）

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

## [2026-07-30] ingest | shaulBespokeNonStationarySolvers2024 Bespoke Non-Stationary Solvers (Shaul et al. 2024, ICML)
- created: `wiki/sources/shaulBespokeNonStationarySolvers2024.md`
- updated: `raw/literature-notes/shaulBespokeNonStationarySolvers2024.md` frontmatter (ingested_to_wiki, wiki_page)

## [2026-07-30] ingest | wangTamingRectifiedFlow2025 Taming Rectified Flow for Inversion and Editing (Wang et al. 2025)
- created: `wiki/sources/wangTamingRectifiedFlow2025.md`
- updated: `raw/literature-notes/wangTamingRectifiedFlow2025.md` frontmatter (ingested_to_wiki, wiki_page)
