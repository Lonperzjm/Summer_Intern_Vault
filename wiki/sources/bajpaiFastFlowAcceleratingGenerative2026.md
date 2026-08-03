---
type: source
title: "FastFlow: Accelerating The Generative Flow Matching Models with Bandit Inference"
aliases: [FastFlow, "Bajpai et al. 2026"]
tags: [flow-matching, solver, adaptive, acceleration, training-free, bandit, inference]
status: active
created: 2026-07-31
updated: 2026-07-31
raw: "[[raw/literature-notes/bajpaiFastFlowAcceleratingGenerative2026]]"
authors: [Divya Jyoti Bajpai, Dhruv Bhardwaj, Soumya Roy, Tejas Duseja, Harsh Agarwal, Aashay Sonsale]
venue: ICLR 2026
year: 2026
arxiv: "2602.11105"
---

# FastFlow: Accelerating The Generative Flow Matching Models with Bandit Inference

> 源页。原文 [arXiv 2602.11105](https://arxiv.org/abs/2602.11105)，ICLR 2026。

## Motivation

FM 推理需要顺序求解 ODE $\frac{dx_t}{dt} = v_\theta(x_t, t)$，每步调用一次大型 DiT/Transformer。50 步 ≈ 50 次昂贵 forward。直接减步（粗网格）对高曲率轨迹损失语义；TeaCache 等缓存方法使用静态阈值，不能很好适应不同轨迹。

FastFlow 的核心问题：**在哪些 timestep 可以不调用模型，一次可以安全跳过几步？**

## Method

### 核心机制一：有限差分 velocity 外推

保存最近两次真实模型预测 $(v_p, v_k)$ 于时间 $(t_p, t_k)$，估计速度变化率：

$$\widehat{\frac{dv}{dt}} = \frac{v_k - v_p}{t_k - t_p}$$

向未来外推：

$$\hat{v}(t_k + \tau) = v_k + \tau \cdot \frac{v_k - v_p}{t_k - t_p}$$

本质假设：**短时间内 velocity 近似线性变化**。相当于从两次 velocity evaluation 中估计"加速度"，比直接复用 $v_k$ 更准确。论文附录将其解释为 Euler solver 上的线性多步更新。

### 核心机制二：UCB Bandit 选择跳步长度

可选 skip length 视为多臂老虎机的 arms，例如 $m \in \{0, 2, 4, 6\}$。每个 timestep $t_k$ 有独立 bandit $B_{t_k}$，采用 UCB 策略：

$$m_k = \arg\max_m \left[ Q_k(m) + \gamma \sqrt{\frac{\log n_k}{N_k(m)}} \right]$$

**Reward 设计**：跳 $m$ 步后调用一次真实模型，比较外推 velocity 与真实 velocity：

$$r_k(m) = \mu m - \ell(\hat{v}_{k+m+1}, v_{k+m+1})$$

- 第一项奖励多跳（加速）
- 第二项惩罚外推误差（保质量）
- 下一次真实模型调用既用于继续生成，也顺便提供 bandit feedback

### 完整推理流程

1. 已知最近两次真实 velocity
2. 当前 timestep 的 bandit 选择 skip length $m$
3. 用有限差分估计 $dv/dt$
4. 线性外推速度推进 $m$ 个中间 timestep（不调用模型）
5. 在跳跃终点调用一次真实 flow model
6. 比较外推 vs 真实 velocity → 计算 reward → 更新 bandit
7. 继续下一段

### 理论结果

在 velocity Lipschitz + 时间二阶导有界假设下，总步数 $T$，被近似集合 $S$：

$$|x^{\text{approx}}_{t_T} - x^{\text{true}}_{t_T}| = O\left(\frac{|S|}{T^3}\right)$$

每多跳一步误差上界线性增加；更细的时间离散降低外推误差。但这只是 latent 数值误差上界，不直接等价于感知质量保证。

## Results

### 图像生成（50-step baseline）

| 配置 | Wall-clock | GenEval | CLIP-IQA | Speedup |
|------|-----------|---------|----------|---------|
| Full 50 | 36.2s | 0.78 | 0.85 | 1× |
| FastFlow-50 | 13.7s | 0.78 | 0.83 | **2.65×** |

更激进配置：FastFlow-25 ≈ 4.5×，FastFlow-10 ≈ 7×（质量下降更明显）。

### 测试覆盖

- 图像生成：BAGEL、FLUX-Kontext、PeRFlow
- 图像编辑：BAGEL、FLUX-Kontext、Step-1X-Edit
- 视频生成：HunyuanVideo
- Benchmark：GenEval、GEdit、VBench 子集

速度—质量 Pareto 曲线通常优于直接减步和 TeaCache。

## 局限性

1. **Cold start**：Bandit 需要约 50–100 个样本探索后才趋于稳定，初始阶段无明显 speedup
2. **非完全无校准**：$\mu$ 需根据首次完整生成的最大 MSE 缩放；生成与编辑使用不同 $\mu$
3. **部分 baseline 不 apples-to-apples**：InstaFlow、PeRFlow 使用不同 SD 系列权重
4. **"per-sample adaptive"名不副实**：bandit 是非 contextual UCB，arm selection 只依赖历史 $(Q, N)$，不以当前 prompt / $x_t$ / curvature 为输入——实质是 dataset-level 的 timestep-dependent skip policy，不是真正 instance-aware

## 关系

### 与已有 wiki 页的关联

- **[[wiki/sources/shaulBespokeNonStationarySolvers2024|BNS Solver]]**：BNS 离线学全局最优离散化（model-specific），FastFlow 在线决定哪些步可以跳（dataset-level adaptive）——前者改 solver 系数，后者改 computation allocation
- **[[wiki/sources/shaulBespokeSolversGenerative2023|Bespoke Solver 2023]]**：同属"不改模型只改求解策略"路线
- **[[wiki/sources/wangTamingRectifiedFlow2025|RF-Solver]]**：RF-Solver 提高每步精度（高阶展开），FastFlow 减少总步数（跳平滑区）——可组合
- **[[wiki/sources/chaTrainingFreeRefinementFlow2026|FDS]]**：FDS 做 spatial refinement（哪里不可信→挪位置），FastFlow 做 temporal skip（哪里平滑→省计算）——正交
- **[[wiki/methods/rectified-flow|Rectified Flow]]**：FastFlow 利用 RF 轨迹接近直线（velocity 变化小）的特性——正是 reflow 的好处让外推更可靠

### 对我的 thesis 的启示

**判断：表面机制相似（都是"自适应跳步"），但解决的问题层面完全不同。不构成直接竞争。**

关键区分：

| 维度 | FastFlow | 我的方向 |
|------|----------|---------|
| 动机 | 加速（省 NFE） | 避 OOD（保质量） |
| 假设 | 速度场处处正确，只是有些步可以省 | 速度场在交叉区本身就错（被平均化） |
| 跳步含义 | 平滑区跳过 = 省算力 | 奇异区跳过 = 避免在错误速度场里积分 |
| 自适应信号 | 外推误差 MSE（事后发现） | divergence / velocity 突变 / 曲率（事前预判） |
| per-sample | 实质是 dataset-level timestep policy | 真正 instance-aware（基于当前 $x_t$） |
| 奇异区行为 | 恢复完整模型计算（但算出来的 velocity 本身就是错的） | 跳过或特殊处理（因为算了也没用） |

**FastFlow 在奇异区的盲点**：当 velocity 被多模态平均化时，FastFlow 会因为外推误差大而选择不跳、老实调用模型——但调用得到的 velocity 本身就是平均化的垃圾。它没有机制识别"这个区域的模型输出不可信"。

**在 related work 中的定位**：归类为"adaptive NFE allocation for acceleration"，与我的"adaptive strategy for OOD avoidance at singularities"是不同层面的问题。

**可借鉴**：
- 有限差分估计 $dv/dt$ 的技巧可以复用——如果 $dv/dt$ 异常大，可能就是奇异区的信号
- Bandit 框架本身不适合我的场景（需要 instance-aware + 事前预判，不能等事后 reward）

## Open Questions

- [ ] FastFlow 的 bandit 在奇异区附近的行为：它会因外推误差大而不跳，但调用模型得到的 velocity 本身就有 bias——这种 bias 能被 FastFlow 的 reward 机制检测到吗？
- [ ] 能否把 FastFlow 的外推误差信号与 FDS 的 divergence 信号结合：前者检测"速度变化快"，后者检测"速度方向不可信"？
- [ ] FastFlow 在编辑任务中的 inversion 精度如何？如果 inversion 路径经过奇异区，跳步外推会不会放大 inversion 误差？
- [ ] contextual bandit 版本（以 $x_t$ 特征为 context）是否能真正实现 instance-aware？这可能是 FastFlow 的自然改进方向

## 出处

- arXiv: [2602.11105](https://arxiv.org/abs/2602.11105)
- 会议: ICLR 2026
- 作者: Divya Jyoti Bajpai, Dhruv Bhardwaj, Soumya Roy, Tejas Duseja, Harsh Agarwal, Aashay Sonsale（IIT Bombay / Amazon）
