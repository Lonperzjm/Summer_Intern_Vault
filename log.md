# Log

> Append-only 时间线。每条 entry 以 `## [YYYY-MM-DD] <op> | <subject>` 起始，便于 `grep "^## \[" log.md | tail`。

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
