---
type: literature-note
citekey: chenBiAnchorInterpolationSolver2026
title: "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"
aliases: ["@chenBiAnchorInterpolationSolver2026"]
authors: "Hongxu Chen, Hongxiang Li, Zhen Wang, Long Chen"
firstAuthor: "Chen"
year: 2026
itemType: preprint
doi: "10.48550/arXiv.2601.21542"
url: "http://arxiv.org/abs/2601.21542"
zotero: "zotero://select/library/items/ES77TRMR"
tags: [literature, computer-science---computer-vision-and-pattern-recognition, computer-science---artificial-intelligence]
status: unread          # unread | reading | read | skimmed | archived
priority: P2            # P0 must-read | P1 important | P2 normal | P3 maybe
my-rating:              # 1-5，读完再填
created: 2026-08-02
updated: 2026-08-02
ingested_to_wiki: true # 由 Claude Code 在 ingest 后改为 true，并写入 wiki_page 链接
wiki_page: "[[wiki/sources/chenBiAnchorInterpolationSolver2026]]"
---

# Bi-Anchor Interpolation Solver for Accelerating Generative Modeling

> [!info] @chenBiAnchorInterpolationSolver2026 · Chen et al. · 2026
> [Open in Zotero](zotero://select/library/items/ES77TRMR) · [DOI](https://doi.org/10.48550/arXiv.2601.21542) · [URL](http://arxiv.org/abs/2601.21542) · [PDF](file:///home/lonper/Zotero/storage/9N95JHK9/Chen%20等%20-%202026%20-%20Bi-Anchor%20Interpolation%20Solver%20for%20Accelerating%20Generative%20Modeling.pdf)

## Abstract

> [!abstract]- Click to expand
> Flow Matching (FM) models have emerged as a leading paradigm for high-fidelity synthesis. However, their reliance on iterative Ordinary Differential Equation (ODE) solving creates a significant latency bottleneck. Existing solutions face a dichotomy: training-free solvers suffer from significant performance degradation at low Neural Function Evaluations (NFEs), while training-based one- or few-steps generation methods incur prohibitive training costs and lack plug-and-play versatility. To bridge this gap, we propose the Bi-Anchor Interpolation Solver (BA-solver). BA-solver retains the versatility of standard training-free solvers while achieving significant acceleration by introducing a lightweight SideNet (1-2% backbone size) alongside the frozen backbone. Specifically, our method is founded on two synergistic components: \textbf{1) Bidirectional Temporal Perception}, where the SideNet learns to approximate both future and historical velocities without retraining the heavy backbone; and 2) Bi-Anchor Velocity Integration, which utilizes the SideNet with two anchor velocities to efficiently approximate intermediate velocities for batched high-order integration. By utilizing the backbone to establish high-precision ``anchors'' and the SideNet to densify the trajectory, BA-solver enables large interval sizes with minimized error. Empirical results on ImageNet-256^2 demonstrate that BA-solver achieves generation quality comparable to 100+ NFEs Euler solver in just 10 NFEs and maintains high fidelity in as few as 5 NFEs, incurring negligible training costs. Furthermore, BA-solver ensures seamless integration with existing generative pipelines, facilitating downstream tasks such as image editing.

## 为什么要读 / 期望

<!-- 在读之前写：你想从这篇里得到什么？哪个 idea 让你点开它？哪个 [[wiki/concepts/...]] 或 [[wiki/methods/...]] 与之相关？ -->

%% begin why-read %%
-
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

<!-- 三到五条要点。这一段 Claude Code 会在 ingest 时直接拿来开场讨论。 -->

%% begin my-summary %%
# 全文总结

## 一句话概括

**BA-solver 冻结原有 Flow Matching 大模型，只训练一个很小的 SideNet；大模型负责在时间区间两端提供可靠速度，SideNet 负责便宜地补出区间内部速度，再用高阶求积完成积分，从而把高质量生成压缩到约 5–10 次大模型调用。**

它处于两类方法中间：

[  
\text{无需训练但步数多的 ODE solver}  
\quad\longleftrightarrow\quad  
\boxed{\text{BA-solver}}  
\quad\longleftrightarrow\quad  
\text{训练昂贵的一两步生成模型}.  
]

论文当前版本为 v3，发表于 ICML 2026。([arXiv](https://arxiv.org/abs/2601.21542 "[2601.21542] Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 1. 研究动机

Flow Matching 生成需要求解 ODE：

[  
\frac{dx_t}{dt}=v_\theta(x_t,t),  
]

从当前时间 (t) 走到 (t-h)：

# [  
x_{t-h}

x_t-\int_{t-h}^{t}v_\theta(x_\tau,\tau),d\tau.  
]

这里最昂贵的是调用大模型 (v_\theta)，一次调用记为一次 NFE。

现有方法有两种主要问题：

- Euler、DPM-Solver 等外推方法，每个区间调用次数少，但大步长时远距离外推误差大；
    
- Heun、Runge–Kutta 等插值方法更准确，但需要在区间内部串行调用多次大模型；
    
- Consistency Model、MeanFlow 等一两步生成方法速度快，但通常需要大规模训练、蒸馏或微调整个 backbone。
    

作者的目标是：

[  
\boxed{  
\text{提高每个区间的积分精度，同时不增加昂贵的 backbone NFE}  
}  
]

([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 2. SideNet：学习局部速度变化

作者在冻结的 FM backbone 旁边增加一个轻量网络 (S_\phi)。

已知当前状态和速度：

[  
x_t,\qquad  
v_t=v_\theta(x_t,t),  
]

SideNet 根据时间偏移 (\Delta t) 预测另一个时刻的速度：

# [  
\boxed{  
\hat v_{t+\Delta t}

v_t+\Delta t,S_\phi(x_t,v_t,t,\Delta t)  
}  
]

可以把

[  
S_\phi(x_t,v_t,t,\Delta t)  
]

理解为近似某种局部的速度变化率。

由于 (\Delta t) 可以是正数或负数，它既能：

- 从当前时刻预测采样方向上的未来速度；
    
- 从后面的时刻回看较早时间的速度。
    

作者将其称为 **Bidirectional Temporal Perception，双向时间感知**。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

SideNet 本身是轻量条件卷积网络：输入包含当前 latent 和 backbone velocity，并用时间、区间大小和类别条件通过 FiLM 调制；主体使用 depthwise-separable convolutional ResBlocks。论文效率实验统计的可训练参数约为 6M，而 backbone 为约 675M。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 3. 为什么需要两个 anchor

只从起点

[  
(x_t,v_t)  
]

预测整个区间时，离起点越远，SideNet 的预测越不准。

原因是实际速度依赖于状态：

[  
v_\theta(x_\tau,\tau),  
]

但 SideNet 是从固定的 (x_t) 出发预测。随着轨迹移动，

[  
x_\tau-x_t  
]

越来越大，会出现 state drift。

因此作者使用两个可靠速度：

[  
v_t,\qquad v_{t-h},  
]

分别作为左右端 anchor：

```text
t                                             t-h
●---------------------|------------------------●
起点 anchor       区间中点                终点 anchor
 从左向右预测 →                 ← 从右向左预测
```

区间前半段使用起点 anchor，后半段使用终点 anchor。这样 SideNet 最大预测距离由

[  
h  
]

降低为

[  
\frac h2.  
]

([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 4. 每个区间的完整采样算法

BA-solver 在一个区间内分成三阶段。

## 阶段一：Forward Probe

已知起点：

[  
x_t,\qquad v_t.  
]

SideNet 从起点 anchor 预测所有求积节点的速度：

# [  
\hat v_\tau^{\mathrm{fwd}}

v_t+(\tau-t)  
S_\phi(x_t,v_t,t,\tau-t).  
]

利用这些预测先积分出一个临时终点：

# [  
x_{t-h}^{\mathrm{pred}}

x_t-h\cdot  
\operatorname{Quadrature}  
\left(  
{\hat v_\tau^{\mathrm{fwd}}}  
\right).  
]

这个终点只是 predictor，不是最终结果。

---

## 阶段二：Backward Refinement

在临时终点调用一次真正的大模型：

# [  
v_{t-h}

v_\theta(x_{t-h}^{\mathrm{pred}},t-h).  
]

由此得到右端 anchor。

对于更靠近终点的积分节点，从右端往回预测：

# [  
\hat v_\tau^{\mathrm{bwd}}

v_{t-h}  
+  
\bigl(\tau-(t-h)\bigr)  
S_\phi  
\left(  
x_{t-h}^{\mathrm{pred}},  
v_{t-h},  
t-h,  
\tau-(t-h)  
\right).  
]

---

## 阶段三：重新积分

靠近左端的节点使用 forward prediction，靠近右端的节点使用 backward prediction：

[  
\hat v_\tau=  
\begin{cases}  
\hat v_\tau^{\mathrm{fwd}},  
&|\tau-t|\leq|\tau-(t-h)|,\[4pt]  
\hat v_\tau^{\mathrm{bwd}},  
&|\tau-t|>|\tau-(t-h)|.  
\end{cases}  
]

最后重新进行高阶求积：

# [  
x_{t-h}

x_t-h\cdot  
\operatorname{Quadrature}  
\left(  
{\hat v_\tau}  
\right).  
]

这才是当前区间接受的最终状态。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 5. Gauss–Lobatto 在这里做什么

推理时作者使用 4-point Gauss–Lobatto quadrature。

在归一化区间 (s\in[0,1]) 上，四个节点约为：

[  
0,\quad0.2764,\quad0.7236,\quad1,  
]

权重比例为：

[  
1:5:5:1.  
]

所以更新近似为：

[  
\boxed{  
x_{t-h}  
\approx  
x_t-\frac h{12}  
\left[  
v_t  
+5\hat v_{\tau_2}^{\mathrm{fwd}}  
+5\hat v_{\tau_3}^{\mathrm{bwd}}  
+v_{t-h}  
\right]  
}  
]

两个端点由大 backbone 提供，两个内部节点由 SideNet 提供。

这正是 Gauss–Lobatto 适合 bi-anchor 的原因：它天然包含两个端点。训练阶段则使用 3-point Gauss–Legendre，推理阶段使用 4-point Gauss–Lobatto。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 6. 为什么每个区间仍然只增加一次 NFE

一个区间的终点 anchor

[  
v_{t-h}  
]

同时也是下一个区间的起点 anchor：

```text
区间 1:  v₁ ------------- v₀.₈
区间 2:                   v₀.₈ ------------- v₀.₆
区间 3:                                      v₀.₆ -------- v₀.₄
```

因此每推进一个新区间，只新增一次终点 backbone evaluation。

此外，最后一个区间不再计算一个无法继续复用的终点 anchor，而是直接返回 Forward Probe 的结果。因此 (N) 个区间可以严格对应 (N) 次 backbone NFE。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 7. SideNet 怎么训练

作者没有让 SideNet 只在完美的真实轨迹状态上训练，而是使用 **chain-based training**。

一条训练链会连续模拟多个求解区间：

[  
x_t  
\rightarrow x_{t-h}  
\rightarrow x_{t-2h}  
\rightarrow\cdots  
]

后一步输入的是前一步 solver 真正产生的、带误差的状态。因此 SideNet 能看到推理时实际会遇到的 error accumulation 和 off-trajectory 状态。

对于每个区间，SideNet预测终点速度：

# [  
v_{t-h}^{\mathrm{pred}}

v_t-hS_\phi(x_t,v_t,t,-h),  
]

冻结的 backbone 在模拟终点上给出 teacher target：

# [  
v_{t-h}

v_\theta(x_{t-h},t-h).  
]

训练损失为：

# [  
\mathcal L_\phi

## \left|  
v_{t-h}^{\mathrm{pred}}

\operatorname{SG}(v_{t-h})  
\right|_2^2.  
]

其中 stop-gradient 保证不更新 backbone。

论文配置中 chain length 为 8；ImageNet-256 训练约 250 iterations，ImageNet-512 约 500 iterations，batch size 为 4096。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 8. 理论分析的核心结论

## 单 anchor

论文将主要误差归因于 SideNet 没有捕捉到的 state drift。简化分析得到：

[  
E_{\mathrm{single}}(h)  
\approx  
\frac12L_{\mathrm{eff}}Ch^2.  
]

所以单 anchor 的局部误差量级为：

[  
O(h^2).  
]

## 双 anchor

双 anchor 将最大时间偏移限制到 (h/2)，分析得到：

[  
E_{\mathrm{bi}}(h)  
\approx  
\frac14L_{\mathrm{eff}}Ch^2  
\approx  
\frac12E_{\mathrm{single}}(h).  
]

也就是说：

[  
\boxed{  
\text{双 anchor 主要把误差系数减半，并没有改变 }O(h^2)\text{ 阶数}  
}  
]

([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

## “Gauss–Lobatto 是七阶”应当谨慎理解

4-point Gauss–Lobatto 对五次及以下多项式积分精确。如果积分节点速度完全准确，它的纯 quadrature scheme error 是：

[  
O(h^7).  
]

但论文附录特别澄清：

[  
\boxed{  
\text{BA-solver 的总误差仍然由 }O(h^2)\text{ 项控制}  
}  
]

因为实际还存在：

- SideNet 速度预测误差；
    
- 状态漂移；
    
- 临时终点误差；
    
- 多步累积误差。
    

所以 BA-solver 不是严格意义上的整体七阶 ODE solver。高阶求积只是使“数值积分公式本身”的误差很小。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 9. 实验结果

实验采用 REPA-enhanced SiT，在 class-conditional ImageNet-256 和 ImageNet-512 上测试。

主要 FID 结果为：

- **ImageNet-256**
    
    - 5 NFE：2.84
        
    - 7 NFE：1.96
        
    - 15 NFE：1.65
        
- **ImageNet-512**
    
    - 5 NFE：5.18
        
    - 7 NFE：2.88
        
    - 15 NFE：1.83
        

对比之下，ImageNet-256、7 NFE 时：

[  
\text{Euler}=7.56,\qquad  
\text{Flow-DPM}=4.03,\qquad  
\text{BA-solver}=1.96.  
]

其优势主要集中在 5–10 NFE 的 few-step 区域。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

论文还报告，在 ImageNet-256 上，BA-solver 以：

[  
8\text{ NFE},\quad250\text{ iterations},\quad6\text{M trainable parameters}  
]

取得 FID 1.85；对比的一两步训练方法通常需要数万到数十万次训练迭代，并优化约 675M 参数。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

在 H800、batch size 256 的测试中：

[  
1\text{ 次 backbone}=4.1837\text{ s},  
]

[  
4\text{ 次 SideNet}=0.2554\text{ s},  
]

[  
\text{积分开销}=0.0112\text{ s}.  
]

论文测得 SideNet 和积分总共带来约 6% 的额外延迟。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 10. 消融实验告诉了什么

7 NFE、ImageNet-256 下：

- Bi-anchor + Gauss–Lobatto：FID 1.96；
    
- 改为 Single-anchor：FID 4.35；
    
- 不使用中间速度：FID 3.54；
    
- 改用 Simpson：FID 1.97；
    
- 一个中间节点：FID 1.89；
    
- 两个中间节点：FID 1.96。
    

最重要的结论是：

[  
\boxed{  
\text{双 anchor 机制远比具体使用哪种 quadrature 更重要}  
}  
]

Gauss–Lobatto 和 Simpson 几乎没有差异，但单 anchor 会造成明显性能下降。有趣的是，一个中间节点的实测 FID 略好于默认的两个节点；作者仍选择两个节点，理由是其理论积分精度更高。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 11. Image editing

由于 BA-solver 仍然是在求解原始 FM ODE，而不是重新训练一个直接 noise-to-image 的一步映射，它保留了 ODE 的可逆结构。

作者先将图像沿 ODE 反演到噪声端，再改变类别条件生成，实现了：

- Dog (\rightarrow) Cat；
    
- Airliner (\rightarrow) Warplane；
    
- Macaw (\rightarrow) Rooster；
    
- Tiger (\rightarrow) Lion。
    

论文展示的编辑过程使用约 10 NFE。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 12. 这篇论文的本质贡献

它真正提出的不是一个纯经典数值求解器，而是：

[  
\boxed{  
\text{学习型局部 velocity interpolator}  
+  
\text{昂贵但可靠的双端 anchor}  
+  
\text{高阶 quadrature}  
}  
]

可以类比成：

- backbone：负责提供少数精确测量点；
    
- SideNet：负责在测量点之间做便宜的 dense interpolation；
    
- Gauss–Lobatto：负责把这些速度组合成状态更新；
    
- anchor reuse：负责把 backbone 调用摊销到下一步。
    

它没有直接学习整个大跨度 flow map，而只学习局部速度如何变化，因此训练成本比一两步蒸馏方法低很多。

---

# 13. 局限性和需要谨慎的地方

**第一，它不是完全 training-free。** Backbone 虽然冻结，但每个新 backbone、数据集或条件机制通常仍需要训练对应的 SideNet。

**第二，实验范围较集中。** 主要验证是 ImageNet class-conditional REPA+SiT；论文没有充分验证大规模 text-to-image、不同架构或复杂 CFG 场景中的泛化。

**第三，NFE 不包含 SideNet。** 论文给出了约 6% wall-clock overhead，但这是特定 H800、batch size 256 下的结果，小 batch 或不同硬件上的比例可能不同。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

**第四，存在 anchor mismatch。** 算法先在临时状态

[  
x_{t-h}^{\mathrm{pred}}  
]

上计算

[  
v_{t-h}=v_\theta(x_{t-h}^{\mathrm{pred}},t-h),  
]

之后又将状态修正为另一个 (x_{t-h})，但缓存并复用的仍是前面计算的 (v_{t-h})。因此下一步的状态和 anchor velocity 并非严格对应；chain-based training 实际上是在帮助网络适应这种误差。这个结论可以直接从采样算法的计算和复用顺序看出。([arXiv](https://arxiv.org/pdf/2601.21542 "Bi-Anchor Interpolation Solver for Accelerating Generative Modeling"))

---

# 最终评价

这篇论文最值得记住的是：

[  
\boxed{  
\text{不要让大模型计算所有积分节点；  
只让它提供可靠端点，再用小模型补内部轨迹。}  
}  
]

它的优势不是严格提高了整个方法的收敛阶，而是通过：

1. 减少 SideNet 的预测距离；
    
2. 使用两个可靠 anchor；
    
3. 让中间速度并行计算；
    
4. 复用终点 anchor；
    
5. 使用高精度 quadrature；
    

显著压低了 few-step 采样中的误差常数。

因此，BA-solver 可以被定位为一种：

> **轻量训练、保持原 FM backbone、面向 5–10 NFE 区域的学习型 predictor–corrector solver。**
%% end my-summary %%

## 与已有 wiki 的关系

<!-- 用 wikilink 把这篇与已有页面挂上钩；空着也行，ingest 时让 Claude Code 补全。 -->

%% begin wiki-links %%
- 概念：[[wiki/concepts/]]
- 方法：[[wiki/methods/]]
- 实体（作者 / 模型 / 机构）：[[wiki/entities/]]
- 基线 / 对比：
%% end wiki-links %%

## 对我的 thesis 的启示

<!-- 是否会让 [[wiki/overview]] 的 working thesis 移动？为什么？ -->

%% begin thesis-implication %%
-
%% end thesis-implication %%

## Open questions / 后续要查的引用

<!-- 从蓝色高亮里捞出来的"待追"清单。 -->

%% begin open-questions %%
- [ ]
%% end open-questions %%

---

> [!tip]- Ingest 触发提示
> 当本文 `status: read` 且 `ingested_to_wiki: false` 时，对 Claude Code 说：
> `ingest raw/literature-notes/chenBiAnchorInterpolationSolver2026.md`
> 它会读取我的高亮 + 总结 + 关系，生成 `wiki/sources/chenBiAnchorInterpolationSolver2026.md`，并把本文件 frontmatter 的 `ingested_to_wiki` 改为 `true`、`wiki_page` 填好。


%% Import Date: 2026-08-02T11:41:16.893+08:00 %%
