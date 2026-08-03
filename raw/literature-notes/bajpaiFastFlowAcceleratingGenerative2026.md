---
type: literature-note
citekey: bajpaiFastFlowAcceleratingGenerative2026
title: "FastFlow: Accelerating The Generative Flow Matching Models with Bandit Inference"
aliases: ["@bajpaiFastFlowAcceleratingGenerative2026"]
authors: "Divya Jyoti Bajpai, Dhruv Bhardwaj, Soumya Roy, Tejas Duseja, Harsh Agarwal, Aashay Sonsale"
firstAuthor: "Bajpai"
year: 2026
itemType: conferencePaper
doi: "10.48550/arXiv.2602.11105"
url: "http://arxiv.org/abs/2602.11105"
zotero:
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---machine-learning]
status: read
priority: P1
my-rating:
created: 2026-07-31
updated: 2026-07-31
ingested_to_wiki: true
wiki_page: "[[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026]]"
---

# FastFlow: Accelerating The Generative Flow Matching Models with Bandit Inference

> [!info] @bajpaiFastFlowAcceleratingGenerative2026 · Bajpai et al. · 2026
> [DOI](https://doi.org/10.48550/arXiv.2602.11105) · [URL](http://arxiv.org/abs/2602.11105) · [PDF](https://arxiv.org/pdf/2602.11105)

## Abstract

> [!abstract]- Click to expand
> Flow matching models produce high-quality outputs but require many sequential neural function evaluations. FastFlow proposes a training-free adaptive sampler that uses finite-difference velocity extrapolation for smooth intervals and a UCB multi-armed bandit to select skip lengths online. Achieves ~2.65× speedup on 50-step baselines with minimal quality degradation across image generation, editing, and video tasks (BAGEL, FLUX-Kontext, HunyuanVideo).

## 为什么要读 / 期望

%% begin why-read %%
- 与我"跨步"想法表面高度重合——需要判断 novelty 是否被抢占
- 导师 7/29 组会指出需排查已有 solver 工作的重复性，FastFlow 是头号嫌疑
%% end why-read %%

## 高亮颜色约定（个人 convention）

> 🟡 **Yellow** = 关键论点 / takeaway
> 🔴 **Red** = 我有异议 / 可疑结论 / 论文改进点
> 🟢 **Green** = 可借鉴的方法 / 公式 / trick
> 🔵 **Blue** = 后续要追溯的引用
> 🟣 **Purple** = 与我 thesis 直接相关
> ⚫ **Gray** = 背景 / 术语定义

## Annotations

%% begin annotations %%
%% end annotations %%

## 我的总结（读完后填）

%% begin my-summary %%
1. 核心机制：有限差分 velocity 外推 + UCB bandit 选跳步长度。平滑区跳过省 NFE，弯曲区恢复完整模型调用。50-step 基线上约 2.65× 加速。
2. "per-sample adaptive"名不副实：bandit 是非 contextual UCB，arm selection 只依赖历史 (Q,N)，不以当前 prompt/x_t/curvature 为输入。实质是 dataset-level 的 timestep-dependent skip policy，不是真正 instance-aware。
3. 与我的方向不冲突：FastFlow 假设速度场处处正确只是有些步可以省；我的问题是速度场在交叉区本身就错（被平均化）。FastFlow 在奇异区会恢复完整模型调用——但调用得到的 velocity 本身就是平均化的垃圾，它没有机制识别这一点。
%% end my-summary %%

## 与已有 wiki 的关系

%% begin wiki-links %%
- 概念：[[wiki/concepts/flow-matching]]、[[wiki/concepts/probability-flow-ode]]
- 方法：[[wiki/methods/rectified-flow]]
- 实体（作者 / 模型 / 机构）：IIT Bombay、Amazon
- 基线 / 对比：TeaCache、InstaFlow、PeRFlow、直接减步
%% end wiki-links %%

## 对我的 thesis 的启示

%% begin thesis-implication %%
- 不构成直接竞争。动机层面完全不同：FastFlow = 加速（省 NFE），我 = 避 OOD（保质量）
- 在 related work 中归类为"adaptive NFE allocation for acceleration"，与我的"adaptive strategy for OOD avoidance at singularities"是不同层面
- 可借鉴：有限差分 $dv/dt$ 异常大 → 可能就是奇异区信号；但 bandit 框架不适合我的场景（需要 instance-aware 事前预判）
%% end thesis-implication %%

## Open questions / 后续要查的引用

%% begin open-questions %%
- [ ] FastFlow 的 bandit 在奇异区附近行为——外推误差大导致不跳，但模型输出本身有 bias，reward 机制能检测到吗？
- [ ] contextual bandit 版本（以 $x_t$ 特征为 context）是否能真正 instance-aware？
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 已 ingest。Wiki 页面：[[wiki/sources/bajpaiFastFlowAcceleratingGenerative2026]]


%% Import Date: 2026-07-31 %%
