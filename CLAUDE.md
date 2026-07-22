# CLAUDE.md — Vault Schema 与工作流契约

> 本文件是 Claude Code 维护本 wiki 的"宪法"。每次会话开始，请先阅读本文件与 `index.md`。
> 方法论原文：[[Karpathy's_Wiki_Method/llm-wiki]] —— 不改写，只引用。

## 1. 四层架构

| 层 | 路径 | 谁能写 | 说明 |
|---|---|---|---|
| Raw（原始资料） | `raw/` | 仅用户 | 不可变。LLM 只读，不修改、不删除（唯一例外：`raw/literature-notes/*.md` 的 `ingested_to_wiki` 与 `wiki_page` 两个 frontmatter 字段，详见 §5.1） |
| Wiki | `wiki/` | LLM 全权 | 由你（Claude Code）维护：创建、更新、重构、删除 |
| Research（我的研究产出） | `research/` | LLM + 用户协同 | LLM 可写但需用户确认，避免单方面改动 thesis/实验记录 |
| Reports（汇报与项目状态） | `reports/` | LLM + 用户协同 | LLM 可生成 draft；更新状态台账和确认版报告需用户确认（详见 §9） |
| Schema | `CLAUDE.md` | 用户主导 | 工作流契约，演化时与用户讨论后再改 |

## 2. 目录契约

```
raw/
  papers/             # arXiv 论文 PDF 或 Web Clipper 抓的 md
  articles/           # 博客、推文、报告
  talks/              # 讲座/视频文字稿
  notes/              # 用户手写的会议/讨论笔记（也是原始输入）
  literature-notes/   # Zotero Integration 渲染输出的文献笔记（含我的高亮+批注）
  worklogs/           # 用户每日原始工作记录（模板 templates/daily-worklog.md）；LLM 只读
  assets/             # Obsidian 自动下载的图片附件

wiki/
  overview.md   # 领域总览，working thesis 演化区
  entities/     # 人、实验室、机构、具名模型（DDPM, FLUX, SD3, …）
  concepts/     # 数学/技术概念（score matching, CFG, rectified flow, …）
  methods/      # text-guided editing 方法族（InstructPix2Pix, Prompt2Prompt, …）
  benchmarks/   # 数据集与评测（TEdBench, MagicBrush, CLIP-T 指标 …）
  sources/      # 每个 raw 资料对应一篇 LLM 总结页，命名与 raw 中文件 slug 对齐
  comparisons/  # query 时生成的对比/分析（沉淀回 wiki，避免淹没在对话）
  synthesis/    # 综述、open problems、跨页矛盾对照

research/
  thesis.md         # 论文核心 thesis 演化
  ideas.md          # 候选 idea 池
  experiments.md    # 实验记录
  related_work.md   # related work 草稿（引用 wiki，不复制）
  outline.md        # paper outline 草稿

reports/            # 汇报层（§9）：LLM 可维护，状态台账与 confirmed 报告需用户确认
  dashboard.md      # 当前状态入口，主要由 Dataview 展示
  state.md          # 任务/Blocker/风险/待决策的唯一状态源
  weekly/           # 周报时间切片（YYYY-Www.md），不是最新状态源
  meetings/         # 面向交流的会议简报，不是实验真相源
  blockers/         # Blocker 报告（YYYY-MM-DD-<slug>.md）
```

## 3. 命名与写作规范

- 文件名：**英文 kebab-case**，如 `classifier-free-guidance.md`、`instructpix2pix.md`
- 中文标题、别名通过 frontmatter 表达
- `wiki/sources/<slug>.md` 的 slug 与 `raw/.../<slug>.{pdf,md}` 对齐（论文以 arXiv ID 或缩写为 slug，如 `2211-09800-instructpix2pix.md`）
- **谨慎使用中英混杂**：行文以中文为主，术语、模型/方法名、代码标识保留英文原文即可；不要把普通叙述写成中英夹杂的碎片（如"这个 approach 很 promising"），也不要滥用可以直接用中文表达的英文词——混杂难看且难读

## 4. Frontmatter 规范（统一字段，便于 Dataview 查询）

所有 wiki 页面顶部使用：

```yaml
---
type: source | entity | concept | method | benchmark | comparison | synthesis
title: 中文/英文皆可
aliases: [别名1, alias-2]
tags: [diffusion, flow-matching, editing]
status: draft | active | stable | stale
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["[[sources/instructpix2pix]]"]   # 仅 wiki 页用，列出该页所依赖的源
---
```

- `status` 含义：`draft` 初稿；`active` 正在阅读相关文献中持续更新；`stable` 暂时稳定；`stale` 已被新源推翻或过时
- 链接一律用 Obsidian wikilink：`[[concepts/score-matching]]`，便于 Graph view 与反链
- **`source` 类型页例外**：source 页本身即「源」，**不用 `sources:` 字段**；改用 `raw:` 指向原始资料（`[[raw/...]]`），并附书目字段 `authors / venue / year / arxiv`。`sources:` 仅用于 `concept / method / entity / benchmark / comparison / synthesis` 等非 source 页，列出其所依赖的 source 页

## 5. Ingest 工作流（用户："ingest <path>"）

1. 读 `raw/...` 中的新资料
2. 与用户用中文讨论 3–5 条关键 takeaway，确认理解无误
3. 在 `wiki/sources/<slug>.md` 写摘要页，必含章节：
   - **Motivation**：作者要解决什么
   - **Method**：核心机制（含必要的公式/伪代码）
   - **Results**：关键实验结论与数字
   - **关系**：与已有 entities / concepts / methods 的关联（用 wikilink）
   - **对我的 thesis 的启示**：是否需要更新 `wiki/overview.md` 中的 working thesis；如需，做 diff 给用户确认
4. 更新涉及的 `entities/`、`concepts/`、`methods/`、`benchmarks/` 页（一次 ingest 通常触及 10–15 页是正常的）
5. 更新 `index.md` 中对应章节
6. append `log.md`：`## [YYYY-MM-DD] ingest | <Title>`，下方列出新建/修改的文件

### 5.1 当 ingest 源是 `raw/literature-notes/<citekey>.md`（Zotero 文献笔记）

这种文件由 Zotero Integration 插件用 `templates/zotero/literature-note.md` 渲染生成，含**用户的高亮、颜色语义、人工批注、读后总结**。处理时要点：

- **优先信任用户的 takeaway**：文件中"我的总结 / 与已有 wiki 的关系 / 对我的 thesis 的启示"几节是用户人读完写下的，比逐条 annotation 更有信息密度，应作为 wiki 总结页的骨架
- **annotation 颜色按约定解读**：🟡 关键论点 / 🔴 异议 / 🟢 可借鉴方法 / 🔵 待追引用 / 🟣 与 thesis 直接相关 / ⚫ 背景。生成 wiki 页时，🟣 的内容必须流入"对我的 thesis 的启示"，🔵 必须流入"待调研方向"
- **`raw/papers/` 中的 PDF 是只读交叉验证源**：当用户的 takeaway 与某条 annotation 看似冲突，或缺少必要细节（公式、数字）时，再去读对应 PDF
- **ingest 完成后回填 frontmatter**：把 `raw/literature-notes/<citekey>.md` 的 `ingested_to_wiki: false` 改为 `true`，并把 `wiki_page` 填上 `"[[wiki/sources/<citekey>]]"`，`updated` 刷新为今天 —— 这是允许 LLM 修改 raw 区的**唯一例外**，因为这两个字段是工作流元数据，不是内容
- **re-import 之后的回填**：Zotero Integration 的 `{% persist %}` 机制保留**正文 5 段**（`why-read` / `my-summary` / `wiki-links` / `thesis-implication` / `open-questions`）和 annotations，但**不保留 frontmatter**。所以 re-import 会把 `status` / `priority` / `my-rating` / `ingested_to_wiki` / `wiki_page` / `created` 重置为模板默认值（其中 `created` 会被刷成当天 importDate，语义上错误）。**当用户在 re-import 之后再次触发 ingest 时**（或用户显式要求修复 frontmatter 时）：Claude 必须自动回填以下三字段：
  - `ingested_to_wiki: true`
  - `wiki_page: "[[wiki/sources/<citekey>]]"`
  - `created`：从 `wiki/sources/<citekey>.md` 的 `created` 读回（那才是真实首次 ingest 日期）

  其余 `status` / `priority` / `my-rating` 三字段不动，由用户自己重设。`updated` 字段刷新为今天。

### 5.2 Refill 工作流（用户："refill <citekey>" 或仅 "refill"）

re-import 已 ingest 过的文献后的快速修复命令。**不是新 ingest**，只回填 frontmatter。

**触发**：
- `refill <citekey>` —— 显式
- `refill` —— 若 `<current_note>` 是 `raw/literature-notes/*.md`，用其 citekey；否则要求用户指明
- `refill <path>` —— 也接受完整路径

**执行步骤**：
1. 校验 `wiki/sources/<citekey>.md` 存在。**不存在则报错**："这篇还没 ingest 过，请改用 `ingest raw/literature-notes/<citekey>.md`"，停止。
2. 读 `raw/literature-notes/<citekey>.md` 当前 frontmatter；读 `wiki/sources/<citekey>.md` 的 `created` 字段
3. 修改 raw 笔记的 4 个字段：
   - `ingested_to_wiki: true`
   - `wiki_page: "[[wiki/sources/<citekey>]]"`
   - `created: <从 wiki source 读回的日期>`
   - `updated: <今天>`
4. **不动** `status` / `priority` / `my-rating`
5. 报告改了什么（简短 diff 形式）

**不写 `log.md`**：refill 是 re-import 后的常规维护，不是知识层事件。

**实现上**：refill 修改 raw 区，是 §5.1 中"raw 只读"规则的同一例外（同四个字段：`ingested_to_wiki` / `wiki_page` / `updated` / `created`）。

## 6. Query 工作流（用户："query: <问题>"）

1. 先读 `index.md` 定位相关页
2. 读相关 wiki 页（按需读 raw 作交叉验证）
3. 综合作答（中文为主），引用页面用 wikilink
4. 如答案具有保留价值（对比表、时间线、新洞察）：
   - 追问用户："要把这个答案归档为 `wiki/comparisons/<slug>.md` 吗？"
   - 同意则归档并 append `log.md`：`## [YYYY-MM-DD] query | <topic>`

## 7. Lint 工作流（用户："lint wiki"）

输出一份**整改清单**（不直接动手），列出：
- 矛盾：跨页冲突的声明
- Stale：被更新的源推翻但未更新的页
- Orphan：无任何反链的孤立页
- Missing：在多页被频繁提及但没有自己的页面的术语
- Broken：失效的 wikilink、frontmatter 字段缺失
- Suggestion：值得新调研的方向 / 可补的源

用户确认整改项后再批量修复，append `log.md`：`## [YYYY-MM-DD] lint | <summary>`

## 8. Research 区规则

- LLM 可读 `research/`，写入前必须征得用户确认
- `related_work.md` 必须用 wikilink 引用 `wiki/`，**不允许**把 wiki 内容复制粘贴进来（保持单一事实源）
- `experiments.md` 由用户主导记录，LLM 仅协助格式化与交叉引用

## 9. Report 工作流（科研汇报体系）

> 完整设计见 `CLAUDE_CODE_REPORTING_SYSTEM_SPEC.md`。核心闭环：`raw/worklogs` 原始事实 → `reports/state.md` 状态 → 周报 / Blocker 汇报 / 组会简报 → 可验收承诺 → 下次汇报逐条对账。

### 9.0 通用约束

- **状态与证据分离**：`raw/worklogs/` 是用户记录的事实证据（LLM 只读）；`reports/state.md` 是任务/Blocker/风险/待决策的**唯一状态源**；`reports/weekly/` 是时间切片；`reports/meetings/` 是交流简报；正式实验结论只归 `research/experiments.md`，周报只能引用或提出待回填项。
- **工作状态**统一使用：`completed` / `in-progress` / `attempted` / `blocked` / `planned` / `dropped`，不允许混用。"阅读了""调试了""思考了"默认不是 `completed`，除非产生明确交付物。
- **结论强度**统一标记：`observation` / `hypothesis` / `conclusion` / `decision` / `unknown`。不得把 observation 自动升级成 conclusion，也不得把 hypothesis 写成事实。
- **事实约束**：不编造实验、指标、时间、commit、路径、导师意见或完成状态；无证据写"待补充"；不得从文件修改时间推断工作完成。"完成"必须同时满足：有交付物、有验收标准、当前证据显示标准已达到。失败实验必须保留但不强行给根因；结论必须带边界；求助项必须可回答（背景/尝试/选项/倾向/希望对方决定什么）；下周计划必须含交付物与验收标准（"继续调研/继续优化"不是合格任务）。
- **单一事实源**：实验事实 → `research/experiments.md`；当前任务状态 → `reports/state.md`；历史周报 → `reports/weekly/`；知识结论 → `wiki/`；Vault 操作历史 → `log.md`。周报可引用但不得成为唯一归档位置。
- **`state.md` ID 规则**：任务 `T-YYYYMMDD-NN`、Blocker `B-YYYYMMDD-NN`、风险 `R-YYYYMMDD-NN`、决策 `D-YYYYMMDD-NN`。已有 ID 永不重编号；关闭项在 Recently Closed 保留至少四周，再由用户确认是否归档。
- **Frontmatter** 以 `templates/{daily-worklog, weekly-report, blocker-report, meeting-brief}.md` 为准（type 分别为 `worklog` / `weekly-report` / `blocker` / `meeting-brief`）。

### 9.1 `report weekly [YYYY-Www]`

从原始记录生成周报并更新状态闭环：

1. 确定周期；未指定时使用最近一个完整周，并明确起止日期。
2. 阅读：对应周期的 `raw/worklogs/*.md`；`reports/state.md`；上一份 confirmed 周报；本周被 worklog 明确引用的 `research/*.md`、代码、配置、日志或图表；必要时读相关 wiki 页（但 wiki 不能替代本周工作证据）。
3. **逐条核对上一期承诺**，不允许未完成任务静默消失。
4. 材料分类为 completed / in-progress / attempted / blocked / planned / dropped。
5. 实验内容区分 observation / hypothesis / conclusion / decision / unknown。
6. 先输出"证据缺口与待确认项"，再生成 draft 周报。
7. 未经用户确认：可写 `reports/weekly/<YYYY-Www>.md` 但 `status` 必须为 `draft`；**不**更新 `state.md`；**不**修改 `research/`。
8. 用户确认后：周报改 `confirmed`；用"下周承诺、Blocker、风险、待决策"更新 `state.md`；发现正式实验记录缺失时只列建议 patch，确认后再改 `research/experiments.md`；append `log.md`：`## [YYYY-MM-DD] report | YYYY-Www confirmed`。

### 9.2 `report blocker <问题或路径>`

1. 读取用户提供的日志、配置和相关 worklog。
2. 生成 `reports/blockers/<YYYY-MM-DD>-<slug>.md` draft，明确区分已知 / 推测 / 未知。
3. 检查是否已有足够最小复现、已尝试方案和具体求助请求。
4. 用户确认后在 `state.md` 登记或更新对应 Blocker ID。
5. 解决后补根因、验证方式、可沉淀资产；用户确认后标 resolved。
6. **紧急事故**（数据误删、服务器故障、凭证泄漏风险等）必须优先建议立即通知相关负责人，不要求先完成完整报告。

### 9.3 `report meeting [date] [group|1on1]`

1. 阅读 `reports/state.md`、最近 confirmed 周报及相关实验记录。
2. 只选最重要的 1–3 项进展、1–2 个问题和需要现场决定的事项。
3. 每个结果同时写明"说明什么"和"不能说明什么"。
4. 生成 `reports/meetings/<date>-<type>.md`；会后由用户填写或提供决定记录。
5. 整理会后决定时，先展示对 `state.md` 和 `research/` 的拟议更新，用户确认后执行。

### 9.4 `report status`

只读检查，不写文件。输出：当前 P0/P1 任务；七天内到期任务；已逾期任务；活跃 Blocker 及持续时间；待导师决策事项；缺少证据或验收标准的任务；建议今天优先处理的第一件事。

### 9.5 `report sync`

对 `raw/worklogs/`、最近周报和 `state.md` 做一致性检查，**只输出整改清单，不直接修改**：周报提到但 state 缺失的活跃任务；state 标 completed 但无证据的任务；到期后无状态更新的任务；已解决但仍标 active 的 Blocker；正式实验结论尚未进入 `research/experiments.md` 的候选项；会议决定尚未转成任务或 decision 的事项。用户确认后才执行批量同步。

## 10. 日志格式

`log.md` 每条形如：

```
## [YYYY-MM-DD] <op> | <subject>
- created: wiki/...
- updated: wiki/...
```

`op` ∈ {`init`, `ingest`, `query`, `lint`, `refactor`, `thesis-update`, `report`}。

`report` op 仅在以下情形写日志：confirmed 周报、已确认的 Blocker 关闭、重大 reporting schema 变更；**draft 不写**。

便于 `grep "^## \[" log.md | tail -10` 快速看最近活动。

## 11. 不变量（Self-check）

每次写完 wiki 后自检：
- [ ] 所有新建页都有完整 frontmatter
- [ ] 所有新建页至少有 1 条入链或在 `index.md` 注册
- [ ] `updated` 字段已刷新
- [ ] `index.md` 与 `log.md` 已同步

每次写完 reports 后追加自检：
- [ ] 报告引用的文件真实存在
- [ ] completed 项有证据与验收标准
- [ ] confirmed 周报与 `reports/state.md` 已同步
- [ ] 不存在未说明的上期承诺消失
- [ ] 未经确认没有修改 `research/`

不满足时回填，再向用户报告完成。
