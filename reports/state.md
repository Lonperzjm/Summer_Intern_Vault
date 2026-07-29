---
type: report-state
updated: 2026-07-28
---

# Research Reporting State

> 本页只维护当前状态。历史过程见 `reports/weekly/`，实验真相见 [[research/experiments]]，Vault 操作历史见 [[log]]。

## Active Commitments

| ID | 优先级 | 任务 | 状态 | 交付物 | 验收标准 | 截止时间 | 证据 | 来源 |
|---|---|---|---|---|---|---|---|---|
| T-20260728-01 | P0 | FM 直线段交叉多模态问题：理论框架 + toy 验证 | in-progress | 奇异点统一框架完善 + 2D/toy 数值验证 | (1) 数值验证 $E(d_{\min})\propto\sqrt{d}$；(2) 三种修正策略 toy 对比 | 8/10 | [[research/notes/2026-07-28-singularity-unified-framework]]、[[research/notes/2026-07-27-high-dim-crossing-probability]] | 用户 7/28 方向确认 |
| T-20260728-02 | P0 | 与导师/师兄确认多模态问题方向 sliver | planned | 明确的 execution 方向（一段话） | 能据此设计第一个正式实验 | 7/29 组会 | — | 用户 7/28 方向确认 |
| T-20260728-03 | P1 | Energy-guidance 线作为副线维护 | planned | 保持文献跟踪，不主动推进实验 | 有新相关论文时 ingest 即可 | 持续 | [[wiki/synthesis/energy-guidance-landscape]] | 用户 7/28 降级 |
| T-20260728-04 | P1 | RF-Inversion 系论文 ingest（editing 直接相关） | planned | 1-2 篇 wiki source 页 | wiki 页完成且链入 overview | 8/3 | — | worklog 7/28 |

## Active Blockers

| ID | 问题 | 严重度 | 已阻塞时间 | 下一动作 | 升级时间 | 详情 |
|---|---|---|---|---|---|---|
| B-20260728-01 | 多模态问题方向的具体 execution sliver 未定（"缩步 vs 跨步 vs spatial shift"三策略选哪个深挖） | medium | 0 天（方向刚确认） | 7/29 组会讨论 | 8/5 前须有结论 | 与 T-02 对应 |

## Risks

| ID | 风险 | 触发条件 | 影响 | 缓解措施 | 状态 |
|---|---|---|---|---|---|
| R-20260728-01 | 暑研时间流逝而选题未落地实验 | 8 月中旬仍无可跑通的 toy experiment | CVPR'27 时间窗口严重收紧 | 本周启动 toy experiment，不等 sliver 完全确认 | active |

## Pending Decisions

| ID | 待决策问题 | 选项 | 我的倾向 | 需要谁决定 | 最晚时间 | 来源 |
|---|---|---|---|---|---|---|
| D-20260728-01 | 多模态问题三种修正策略选哪个作为 thesis 主攻 | A. 缩步（adaptive step-size）B. 跨步（新思路）C. spatial shift（FDS 路线）D. 组合 | B（跨步）novelty 最高，但需验证精度代价 | 导师/自己实验后判断 | 8/10 | [[research/notes/2026-07-28-singularity-unified-framework]] |

## Recently Closed

| ID | 项目 | 结果 | 关闭日期 | 证据/去向 |
|---|---|---|---|---|
| — | Bridge-SDE editing 全方向 | 正式关闭：三次 sweep 三撞 + 用户 7/28 确认舍弃 | 2026-07-28 | [[research/ideas]] Killed 节；[[wiki/synthesis/bridge-sde-editing-landscape]] |
| — | Thesis v0.1（bridge-SDE 施力点） | 正式关闭：方向已转 | 2026-07-28 | [[research/thesis]] v0.1 标注 |
