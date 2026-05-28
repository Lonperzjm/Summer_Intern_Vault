---
title: Index
updated: 2026-05-28
---

# Index

> Vault 的内容目录。Claude Code 在每次 ingest 后自动维护本文件。
> 安装 [Dataview] 后，下面的代码块会渲染成动态表格；未安装则显示为代码块也可读。

## Overview

- [[wiki/overview]] —— 领域总览与 working thesis

## 📚 阅读清单（raw/literature-notes）

> **字段速查**：
> - `status`: `unread` / `reading` / `read` / `skimmed` / `archived`
> - `priority`: `P0` 必读 / `P1` 重要 / `P2` 普通 / `P3` 也许
> - `my-rating`: 1–5（读完再填）
> - `wiki ✓` 显示是否已 ingest 到 `wiki/sources/`

### 🔥 优先阅读（P0 / P1 且未读完）

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  firstAuthor + " " + year AS "Ref",
  status AS "Status",
  priority AS "Pri"
FROM "raw/literature-notes"
WHERE (priority = "P0" OR priority = "P1") AND status != "read" AND status != "archived"
SORT priority ASC, updated DESC
```

### 📖 在读

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  firstAuthor + " " + year AS "Ref",
  priority AS "Pri",
  updated AS "Updated"
FROM "raw/literature-notes"
WHERE status = "reading"
SORT priority ASC, updated DESC
```

### ⭐ 高分已读（`my-rating ≥ 4`）

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  firstAuthor + " " + year AS "Ref",
  my-rating AS "⭐",
  choice(ingested_to_wiki, "✅", "❌") AS "wiki ✓"
FROM "raw/literature-notes"
WHERE my-rating != null AND number(my-rating) >= 4
SORT number(my-rating) DESC, updated DESC
```

### 📋 全部 literature notes

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  firstAuthor + " " + year AS "Ref",
  status AS "Status",
  priority AS "Pri",
  my-rating AS "⭐",
  choice(ingested_to_wiki, "✅", "❌") AS "wiki ✓",
  updated AS "Updated"
FROM "raw/literature-notes"
SORT priority ASC, updated DESC
```

---

## Sources（已 ingest 的资料）

```dataview
TABLE title, status, updated
FROM "wiki/sources"
SORT updated DESC
```

## Entities（人 / 机构 / 具名模型）

```dataview
TABLE title, tags, updated
FROM "wiki/entities"
SORT title ASC
```

## Concepts（数学与技术概念）

```dataview
TABLE title, status, updated
FROM "wiki/concepts"
SORT updated DESC
```

## Methods（text-guided editing 方法族）

```dataview
TABLE title, status, updated
FROM "wiki/methods"
SORT updated DESC
```

## Benchmarks（数据集与评测）

```dataview
TABLE title, updated
FROM "wiki/benchmarks"
SORT title ASC
```

## Comparisons（沉淀的对比与分析）

```dataview
TABLE title, updated
FROM "wiki/comparisons"
SORT updated DESC
```

## Synthesis（综述 / open problems）

```dataview
TABLE title, status, updated
FROM "wiki/synthesis"
SORT updated DESC
```

## Research（我自己的产出）

- [[research/thesis]]
- [[research/ideas]]
- [[research/experiments]]
- [[research/related_work]]
- [[research/outline]]

---

## Stale 与 Draft 提醒

```dataview
TABLE file.folder AS folder, status, updated
FROM "wiki"
WHERE status = "stale" OR status = "draft"
SORT updated ASC
```
