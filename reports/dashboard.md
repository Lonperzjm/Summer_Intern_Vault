---
type: report-dashboard
title: Reporting Dashboard
updated: 2026-07-21
---

# Reporting Dashboard

> 汇报层总入口。任务/Blocker/风险/待决策的**唯一状态源**是 [[reports/state]]（普通 Markdown 表格，Dataview 无法可靠解析，故不在本页复制成第二份数据库）。

## 快速入口

- [[reports/state]] —— 当前任务、Blocker、风险与待决策事项
- 最新 confirmed 周报：暂无（从下一次 `report weekly` 开始生成，不回填历史）
- 下一次会议简报：暂无
- [[research/experiments]] —— 正式实验结论的唯一来源
- [[research/ideas]] —— 候选 idea 池
- [[research/thesis]] —— thesis 演化

## 最近周报（最多 8 份）

```dataview
TABLE period, status, start, end, updated
FROM "reports/weekly"
WHERE type = "weekly-report"
SORT period DESC
LIMIT 8
```

## Active Blockers

```dataview
TABLE title, severity, project, opened, updated
FROM "reports/blockers"
WHERE type = "blocker" AND status = "active"
SORT updated DESC
```

## 最近会议简报（最多 8 份）

```dataview
TABLE meeting, date, status, projects
FROM "reports/meetings"
WHERE type = "meeting-brief"
SORT date DESC
LIMIT 8
```
