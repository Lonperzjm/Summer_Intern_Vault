# Log

> Append-only 时间线。每条 entry 以 `## [YYYY-MM-DD] <op> | <subject>` 起始，便于 `grep "^## \[" log.md | tail`。

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
