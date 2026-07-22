# Summer_Intern_Vault

> 暑研知识库：Diffusion / Flow Matching · Text-Guided Image Editing
> 基于 [Karpathy 的 LLM-Wiki 方法](Karpathy's_Wiki_Method/llm-wiki.md) 构建。

## 这是什么

一个由 **Claude Code 维护、Obsidian 浏览** 的个人知识库。Karpathy 的核心思想：你负责**找资料、提问、做判断**；LLM 负责**总结、归档、维护交叉引用**。Obsidian 是 IDE，LLM 是程序员，wiki 是被持续编译的代码库。

三层结构：
- `raw/`：你丢进去的原始资料（论文、博客、笔记），**不可变**；其中 `raw/worklogs/` 是你每天写的原始工作记录
- `wiki/`：Claude Code 写的总结、概念页、方法页、对比 —— 你只读
- `research/`：你自己的论文进展（thesis、实验、outline），LLM 协助但不擅自改
- `reports/`：科研汇报层（状态台账 + 周报 + Blocker + 组会简报），LLM 生成 draft、你确认后定稿；入口见 [reports/dashboard.md](reports/dashboard.md)

工作流契约写在 [CLAUDE.md](CLAUDE.md)，每次会话 Claude Code 会自动加载。

---

## 快速上手（5 步）

1. **加资料**：把 PDF / Web Clipper 抓的 md 放进 `raw/papers/` 或 `raw/articles/`
2. **Ingest**：在本目录开 Claude Code，说 `ingest raw/papers/<filename>`
3. **浏览**：在 Obsidian 中打开 `wiki/sources/<slug>.md`，跟着 wikilink 跳转，开 Graph view 看连接
4. **提问**：直接问 Claude Code，比如 `query: rectified flow 与 score-based diffusion 在编辑任务上的本质区别？`；有价值的回答让它归档到 `wiki/comparisons/`
5. **每周一次 Lint**：说 `lint wiki`，让 Claude Code 找矛盾、孤立页、stale 内容并修

常用 prompt：
```
ingest raw/papers/2211-09800-instructpix2pix.pdf
ingest 最近 3 篇 raw/articles/
query: 现有 text-guided editing 方法在保真度-可控性 trade-off 上的对比？归档为 comparison
update working thesis based on 最近一周 ingest 的源
lint wiki
```

---

## Obsidian 推荐配置

### 核心插件（Settings → Core plugins）
开启：Backlinks、Outgoing links、Tag pane、Templates、Graph view、Outline、Page preview。

### 社区插件（Settings → Community plugins → Browse）
| 插件 | 用途 | 必装？ |
|---|---|---|
| **Dataview** | 让 `index.md` 按 frontmatter 动态生成列表 | ✅ |
| **Templater** | 与 `templates/` 配合，新建页一键套模板 | 推荐 |
| **Marp** | 把 wiki 内容直接导出组会 slide | 可选 |
| **Image converter / Local images** | 处理 Web Clipper 下来的本地图片 | 可选 |

> **Web Clipper** 是浏览器扩展（不是 Obsidian 插件），从 Chrome/Firefox Web Store 装即可。配置 output folder 指向本 Vault 的 `raw/articles/` 或 `raw/papers/`。

### 设置
- `Settings → Files and links → Default location for new attachments`：选 **In the folder specified below**，路径填 `raw/assets/`
- `Settings → Files and links → New link format`：选 **Relative path to file** 或 **Shortest path**（保持你习惯）
- `Settings → Hotkeys`：搜索 `Download attachments for current file`，绑定到 `Ctrl+Shift+D`。Web Clipper 抓完后按一下，把所有图片本地化到 `raw/assets/`
- `Settings → Appearance → Graph view`：在 Graph 设置里按 `frontmatter: type` 着色，能一眼看出 entity / concept / method / source 的分布与孤立节点

### Dataview 示例（已写入 `index.md`）
```dataview
TABLE status, updated FROM "wiki/concepts" SORT updated DESC
```

---

## 科研汇报体系（reports/）

日常闭环：你每天在 `raw/worklogs/` 写原始记录（模板 `templates/daily-worklog.md`）→ Claude 在周报时统一归纳 → 状态沉淀到 `reports/state.md` → 下次汇报逐条对账。规则详见 [CLAUDE.md](CLAUDE.md) §9。

```text
# 用户每天在 Obsidian 中写
raw/worklogs/2026-07-20.md

# 查看当前状态，不写文件
report status

# 问题卡住时
report blocker 训练后期出现 NaN，日志见 runs/exp-017/train.log

# 周五生成周报草稿
report weekly 2026-W30

# 用户校正并确认后
确认这份周报；用其中的下周承诺和 blockers 更新 state

# 组会前
report meeting 2026-07-27 group

# 每周做一次一致性检查
report sync
```

要点：
- `reports/state.md` 是任务/Blocker/风险/待决策的**唯一状态源**；周报是时间切片，不是最新状态
- Claude 只能写 draft；`confirmed` 周报、state 更新、`research/` 改动都需要你确认
- 正式实验结论只归 `research/experiments.md`；周报只引用

---

## 给 Claude Code 的 prompt 范式

| 意图        | 示例                                                                     |
| --------- | ---------------------------------------------------------------------- |
| Ingest 单源 | `ingest raw/papers/xxx.pdf`                                            |
| 批量 ingest | `ingest 全部未处理的 raw/papers/*`                                           |
| 查询 + 归档   | `query: <问题>，答完归档为 comparison`                                         |
| 更新 thesis | `根据 wiki/synthesis 的最新内容更新 wiki/overview.md 的 working thesis，diff 给我看` |
| 健康检查      | `lint wiki`                                                            |
| 调整 schema | `我们以后把每个 method 页都加一个"failure modes"小节，更新 CLAUDE.md`                   |

---

## 版本控制（可选但强烈推荐）

```bash
git init
echo ".obsidian/workspace.json" >> .gitignore
echo ".obsidian/workspace-mobile.json" >> .gitignore
git add . && git commit -m "init: scaffold vault per Karpathy wiki method"
```

`raw/` 和 `wiki/` 都进 git，整个知识演化过程都有时间机器。

---

## 文件入口

- [CLAUDE.md](CLAUDE.md) —— 工作流契约（必读）
- [index.md](index.md) —— 内容索引
- [log.md](log.md) —— 时间线
- [wiki/overview.md](wiki/overview.md) —— 领域总览与 working thesis
- [Karpathy's_Wiki_Method/llm-wiki.md](Karpathy's_Wiki_Method/llm-wiki.md) —— 方法论原文
