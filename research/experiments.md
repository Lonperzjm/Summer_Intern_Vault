---
type: research
title: 实验记录
status: active
created: 2026-05-05
updated: 2026-08-03
---

# Experiments

> 本页只做实验索引与结论范围摘要。完整设置、指标和边界以链接的 canonical report 为准；新报告使用 [[templates/experiment-report]]。

## 实验目录

| Experiment ID | Canonical report | 文档状态 | 实验状态 | 结论范围 |
|---|---|---|---|---|
| EXP-20260802-reject-and-skip-crossing-toy | [[research/experiments/2026-08-02-reject-and-skip-toy-report]] | confirmed | completed | 二维 oracle toy 支持结构化 coupling change；不能外推到真实神经速度场 |
| EXP-20260803-official-1rf-solver-diagnostics | [[research/experiments/2026-08-03-official-1rf-solver-diagnostics]] | confirmed | completed | 完成真实 1-RF 数值基线与诊断；reject-and-skip 机制和分布收益仍未知 |

## 动态视图

```dataview
TABLE experiment_id, date, status, experiment_status, project
FROM "research/experiments"
WHERE type = "experiment-report"
SORT date DESC
```
