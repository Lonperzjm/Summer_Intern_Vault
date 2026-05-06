---
title: Index
updated: 2026-05-05
---

# Index

> Vault 的内容目录。Claude Code 在每次 ingest 后自动维护本文件。
> 安装 [Dataview] 后，下面的代码块会渲染成动态表格；未安装则显示为代码块也可读。

## Overview

- [[wiki/overview]] —— 领域总览与 working thesis

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
