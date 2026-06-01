1. 阅读须知（简要版，详见由ai编写的[[GUIDANCE]]）：
	1. 体系结构：
		* `./raw` 是由人类手写的笔记，或者是外界找到的文件，如论文pdf等
		* `./research` 是人类的研究进展（thesis、实验、outline），目前置空
		* `./wiki` 是ai根据上面内容总结整理的知识库，可以但是不建议人类直接阅读。
	2. `./wiki`使用方法：
		* 对ai使用 `query` 语句，如：`query 请总结图像生成的几条主要技术路径，比较特点与优劣`，ai会查询`./wiki` 并回答
		* 注意：若并非使用的 **claude code** 工具，可以将[[CLAUDE]]的名称改为其他ai工具规定的约束格式。
		* 注意：如果不是使用 **obsidian** markdown阅读器，可能会有格式问题。但是大部分阅读器应该可以正常阅读。


2. 工作记录(简要版，详见由ai编写的[[log]])：
	* 5.1-5.7 构建zotero与obsidian环境，整理论文，构建Summer_Intern_Vault。学习U-Net、VAE、crossattn等基本知识
	* 5.8-5.11 DDPM + DDIM bg+NON-MARKOVIAN FORWARD PROCESSES 。形成 [[raw/literature-notes/hoDenoisingDiffusionProbabilistic2020|hoDenoisingDiffusionProbabilistic2020]] 阅读笔记，半完成 [[raw/literature-notes/songDenoisingDiffusionImplicit2022|songDenoisingDiffusionImplicit2022]] 阅读笔记，`ingest ddpm` 
	* 5.11-5.17 完成DDIM, 完成[[raw/literature-notes/songDenoisingDiffusionImplicit2022|songDenoisingDiffusionImplicit2022]] 阅读笔记，读 score-based SDEs abstract+intro+conclusion, 半完成[[songScoreBasedGenerativeModeling2021]] 笔记。`ingest ddim`
	* 5.18-5.24 完成score-based SDEs，Flow Matching。完成[[raw/literature-notes/songScoreBasedGenerativeModeling2021|songScoreBasedGenerativeModeling2021]]、[[raw/literature-notes/lipmanFlowMatchingGenerative2023|lipmanFlowMatchingGenerative2023]] 阅读笔记，`ingest score-basedmodel & flowmatching`。建立基本的概率-采样概念理解。
	* 5.25-5.31 完成**Rectified Flow**、**Latent Diffusion Models**、**ControlNet**、**SDEdit**、**Denoising Diffusion Bridge Models**。完成 [[raw/literature-notes/liuFlowStraightFast2022a|liuFlowStraightFast2022a]]、[[raw/literature-notes/rombachHighResolutionImageSynthesis2022|rombachHighResolutionImageSynthesis2022]]、[[raw/literature-notes/zhangAddingConditionalControl2023|zhangAddingConditionalControl2023]]、[[raw/literature-notes/mengSDEditGuidedImage2022|mengSDEditGuidedImage2022]]、[[raw/literature-notes/zhouDenoisingDiffusionBridge2023|zhouDenoisingDiffusionBridge2023]] 阅读笔记，逐篇 `ingest`。由 DDBM 引出 **bridge SDE vs bridge ODE** 视角，初步把 bridge-SDE 上的编辑作为候选 thesis 方向（详见 [[research/thesis]]、[[wiki/synthesis/bridge-sde-editing-landscape]]）。
	* 6.1-6.7 （计划）完成**DDIM**，**Stochastic Interpolants (2303.08797)（略读）**，**Diffusion Bridge or Flow Matching? (2509.24531)** 确认选题，`ingest`对应文章
3. `./wiki`重要内容(简要版，详见由ai编写的[[index]])
	* **研究者 / 机构 / 具名模型**（`entities/`）：领域内关键人物、所属机构与代表性模型的档案
	* **核心概念**（`concepts/`）：数学与技术概念（生成/采样的原理、训练目标、引导与条件注入等）
	* **方法 / 模型族**（`methods/`）：各代生成方法与模型族的机制、流程与适用场景
	* **数据集与评测**（`benchmarks/`）：常用数据集与评测指标的定义、方向与局限
	* **源摘要页**（`sources/`）：每篇已 ingest 文献对应一页 LLM 摘要，对齐 raw 笔记
	* **领域总览与 working thesis**（`synthesis/` → [[wiki/overview]]）：领域脉络梳理与持续演化的核心论点