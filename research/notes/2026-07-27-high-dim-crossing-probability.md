---
type: research-note
title: 高维空间中 RF 路径交叉概率分析
tags: [rectified-flow, high-dimensional, crossing-probability, original-analysis]
status: active
created: 2026-07-27
updated: 2026-08-02
triggered_by: ["[[wiki/sources/2502.17436-towards-hierarchical-rectified-flow]]"]
---

# 高维空间中 RF 路径交叉概率分析

> 原创分析。由阅读 [[wiki/sources/2502.17436-towards-hierarchical-rectified-flow|HRF]] 触发。

## 核心问题与边界

HRF 的一个动机是允许路径携带多模态速度并发生交叉。这里先问一个更窄的几何问题：在**端点独立、近似均匀随机**的简化模型中，两条有限直线段在高维空间里接近的概率有多大？

这个 toy 几何问题不等价于真实 Flow Matching 中的 conditional velocity multimodality。后者取决于 $V\mid X_t=x$ 的条件分布，不要求有限训练样本中的两条线段精确相交。真实图像 latent、noise--data coupling 和数据流形也不满足均匀随机端点假设。

## 分析

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785140429486.webp]]

如上图所示，在简化的两路径图景中，如果两条插值路径相交或非常接近，局部条件速度可能呈多模态，而 marginal velocity 回归得到其平均值。离散积分若试探到该区域，后续速度可能把轨迹带向低密度位置。这一采样机制另见 [[research/notes/2026-07-28-singularity-unified-framework]]。

作为 sanity check，考虑 $d \gg 1$、端点在 $[0,1]^d$ 内独立随机的两条线段，其最小距离 $d_{\min}$：

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785238406195.webp]]

$$E(d_{\min}) \propto \sqrt{d}, \quad D(d_{\min}) \propto 1$$

数值实验中 $d_{\min}$ 在该简化设定下可近似呈集中分布。维度增大时，随机线段精确或近似相交的概率快速降低。

![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785207600516.webp]]
![[zhangHIERARCHICALRECTIFIEDFLOW2025plus-1785147289964.webp]]

## 当前能支持的结论

- **可以说**：在独立均匀随机端点的 toy 几何模型中，高维线段近交叉是低概率事件；因此“任意两条随机有限样本路径的几何交叉”不应未经验证就被当作真实图像 FM 的主要失败来源。
- **不能据此说**：真实 Flow Matching 的速度多模态很少，或 HRF 必然不经济。条件速度平均不要求有限样本线段精确相交，HRF 也可能处理更一般的 posterior velocity multimodality。
- 对 HRF 的成本收益判断必须依赖真实生成实验、条件速度统计和 NFE/质量比较，而不能只由该 toy 推导得出。

## 后续延伸

→ [[research/notes/2026-07-28-singularity-unified-framework|奇异点统一框架]]（将此分析与 FDS 结合）
