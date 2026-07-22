---
type: report-state
updated: 2026-07-21
---

# Research Reporting State

> 本页只维护当前状态。历史过程见 `reports/weekly/`，实验真相见 [[research/experiments]]，Vault 操作历史见 [[log]]。

## Active Commitments

| ID | 优先级 | 任务 | 状态 | 交付物 | 验收标准 | 截止时间 | 证据 | 来源 |
|---|---|---|---|---|---|---|---|---|

## Active Blockers

| ID | 问题 | 严重度 | 已阻塞时间 | 下一动作 | 升级时间 | 详情 |
|---|---|---|---|---|---|---|

## Risks

| ID | 风险 | 触发条件 | 影响 | 缓解措施 | 状态 |
|---|---|---|---|---|---|

## Pending Decisions

| ID | 待决策问题 | 选项 | 我的倾向 | 需要谁决定 | 最晚时间 | 来源 |
|---|---|---|---|---|---|---|

## Recently Closed

| ID | 项目 | 结果 | 关闭日期 | 证据/去向 |
|---|---|---|---|---|

---

## ⚠️ 待用户确认迁移项（初始化草案 · 2026-07-21）

> 以下条目由 Claude 根据 `README.md` 工作记录、[[research/ideas]]、[[research/thesis]] 推断，**全部为 `needs-confirmation`，不是事实**。
> 用户逐条确认（或修正）后，才移入上方正式状态表并分配正式 ID；否决的条目直接删除。
> 本节清空后删除整节。

### 候选 P0 研究目标（needs-confirmation）

- **拟 T-20260721-01**：把 energy-guidance「三段全红」sweep 地图（①②③ 段落点 + FMPS/FlowChef/TtfDiffusion/DICE/GradOPS 证据）带回师兄/导师，由其用非公开在研线信息确定具体 execution sliver。
  - 依据：[[research/ideas]] 2026-06-29 更新的「下一步」；红海三铁律唯一存活项 = 坐导师在研线。
  - 待用户补：交付形式（口头 brief / 一页纸？）、与师兄约定的时间。

### 候选活跃实验（needs-confirmation）

- 无。`research/experiments.md` 仍为空模板；[[research/ideas]] 记「第一个实验设计待 sliver 定」。
  - 请确认：当前确实没有任何在跑/在设计的实验？若有，属未记录状态，需补录。

### 候选 Blocker（needs-confirmation）

- **拟 B-20260721-01**：研究方向（execution sliver）未定，依赖师兄/导师的非公开信息；armchair carve 已三次 sweep 三次撞车，独立推进路径已被证明无效。
  - 严重度候选：high（阻塞 P0 与第一个实验）。
  - 待用户补：上次与师兄沟通日期、下次可触达时间。

### 候选风险（needs-confirmation）

- **拟 R-20260721-01**：暑研时间流逝而选题未定；用户目标为 2026-12 申请季前有顶会在投（主目标 CVPR'27），留给「定题 → 实验 → 成文」的窗口持续收窄。
  - 触发条件候选：若 8 月中旬仍无确定 sliver。

### 候选待决策事项（needs-confirmation）

- **拟 D-20260721-01**：energy-guidance execution sliver 具体定在哪一段（需师兄/导师决定，对应上述 Blocker）。
- **拟 D-20260721-02**：[[research/thesis]] v0.1（bridge-SDE 方向）自 2026-06-18 起标「🔁 暂挂复审」——是正式关闭（移入 Recently Closed）还是继续挂起？方法级 idea 已在 [[research/ideas]] Killed 中记死，但 thesis 文件本身未关闭。

### 已失效但尚未正式关闭的旧方向（needs-confirmation）

- bridge-SDE editing 全家桶、「脱离 diffusion 框架」理论方向：已在 [[research/ideas]] Killed 中带证据关闭，**无需迁移**；仅 thesis v0.1 的挂起状态待上面 D-02 决定。
