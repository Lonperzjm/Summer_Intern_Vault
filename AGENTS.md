# AGENTS.md — Codex Vault 工作契约

> 本文件是 Codex 进入此 Vault 时自动加载的入口。Vault 的完整、权威工作流契约位于 `CLAUDE.md`；不要在这里复制整份契约，以免两份规则随演化而产生漂移。

## 每次会话开始

在执行任何任务或修改任何文件之前：

1. 完整阅读根目录的 `CLAUDE.md`。
2. 阅读根目录的 `index.md`，了解当前知识结构。
3. 根据任务需要检查相关页面及 `log.md` 的近期记录。

除非用户在当前对话中明确给出更高优先级的指示，否则必须遵守 `CLAUDE.md` 的全部章节，包括目录契约、frontmatter、命名、Ingest、Refill、Query、Lint、Research、Report、日志和写后自检规则。

## Codex 适配约定

- 将 `CLAUDE.md` 中对“Claude Code”或“Claude”的行为要求视为对当前 Codex agent 的要求。
- `CLAUDE.md` 仍是 Vault schema 与工作流的唯一事实源；本文件只负责让 Codex 自动发现并加载它。
- `CLAUDE.md` 中出现的文件名、目录名、命令触发词和数据格式保持原义，不因 agent 名称不同而改写。
- 若本文件与 `CLAUDE.md` 对 Vault 内容规则的表述不一致，以 `CLAUDE.md` 为准；若用户当前指令与二者冲突，先指出冲突及影响，再按用户明确决定执行。
- 不要仅因为新增了 Codex 支持而修改或删除 `CLAUDE.md`。

## 必须优先守住的边界

- `raw/` 默认只读、不可删除；仅允许按 `CLAUDE.md` §5.1/§5.2 修改 `raw/literature-notes/*.md` 指定的工作流 frontmatter 字段。
- `wiki/` 可由 Codex 按契约创建、更新、重构或删除。
- 写入 `research/` 前必须获得用户确认。
- `reports/` 可生成 draft；确认版报告、状态台账及相关正式同步必须按 `CLAUDE.md` §9 获得用户确认。
- 修改 Vault schema 或工作流契约前必须与用户讨论并获得确认。
- `lint wiki`、`report status` 和 `report sync` 默认只检查并输出清单，不直接修改文件。
- 不编造来源、实验结果、指标、日期、状态、路径、提交记录或他人意见；证据不足时明确写“待补充”或标记为未知。
- 不覆盖或清理用户已有的未提交改动；遇到与任务重叠的改动时先检查并保留用户内容。

## 写入后的最低检查

- 修改 `wiki/` 后执行 `CLAUDE.md` §11 的 wiki 自检，并同步需要更新的 `index.md`、`log.md` 与 `updated` 字段。
- 修改 `reports/` 后执行 `CLAUDE.md` §11 的 reports 自检。
- 完成任务时简要报告创建、修改的文件，以及任何仍需用户确认的事项。
