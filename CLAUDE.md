# CLAUDE.md — Vault Schema 与工作流契约

> 本文件是 Claude Code 维护本 wiki 的"宪法"。每次会话开始，请先阅读本文件与 `index.md`。
> 方法论原文：[[Karpathy's_Wiki_Method/llm-wiki]] —— 不改写，只引用。

## 1. 三层架构

| 层 | 路径 | 谁能写 | 说明 |
|---|---|---|---|
| Raw（原始资料） | `raw/` | 仅用户 | 不可变。LLM 只读，不修改、不删除（唯一例外：`raw/literature-notes/*.md` 的 `ingested_to_wiki` 与 `wiki_page` 两个 frontmatter 字段，详见 §5.1） |
| Wiki | `wiki/` | LLM 全权 | 由你（Claude Code）维护：创建、更新、重构、删除 |
| Research（我的研究产出） | `research/` | LLM + 用户协同 | LLM 可写但需用户确认，避免单方面改动 thesis/实验记录 |
| Schema | `CLAUDE.md` | 用户主导 | 工作流契约，演化时与用户讨论后再改 |

## 2. 目录契约

```
raw/
  papers/             # arXiv 论文 PDF 或 Web Clipper 抓的 md
  articles/           # 博客、推文、报告
  talks/              # 讲座/视频文字稿
  notes/              # 用户手写的会议/讨论笔记（也是原始输入）
  literature-notes/   # Zotero Integration 渲染输出的文献笔记（含我的高亮+批注）
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
```

## 3. 命名规范

- 文件名：**英文 kebab-case**，如 `classifier-free-guidance.md`、`instructpix2pix.md`
- 中文标题、别名通过 frontmatter 表达
- `wiki/sources/<slug>.md` 的 slug 与 `raw/.../<slug>.{pdf,md}` 对齐（论文以 arXiv ID 或缩写为 slug，如 `2211-09800-instructpix2pix.md`）

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

## 9. 日志格式

`log.md` 每条形如：

```
## [YYYY-MM-DD] <op> | <subject>
- created: wiki/...
- updated: wiki/...
```

`op` ∈ {`init`, `ingest`, `query`, `lint`, `refactor`, `thesis-update`}。

便于 `grep "^## \[" log.md | tail -10` 快速看最近活动。

## 10. 不变量（Self-check）

每次写完 wiki 后自检：
- [ ] 所有新建页都有完整 frontmatter
- [ ] 所有新建页至少有 1 条入链或在 `index.md` 注册
- [ ] `updated` 字段已刷新
- [ ] `index.md` 与 `log.md` 已同步

不满足时回填，再向用户报告完成。
