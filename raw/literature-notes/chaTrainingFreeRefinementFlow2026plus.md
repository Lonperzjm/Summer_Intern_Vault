如[[zhangHIERARCHICALRECTIFIEDFLOW2025plus]]所言，我们的分位相交线段很少。三相交应该更少才对。故只考虑双相交线段。我们称相交出为奇异点，或者多模态点。

直线最小的距离为$d(t)_{min}$,离散步长为$\Delta = v\delta t$,模型的表达能力为$l$,代表模型不能良好拟合超过此尺度的转向![[chaTrainingFreeRefinementFlow2026plus-1785231261637.webp]]

可以估计$\Delta \approx 100l$,即模型的表达能力远远好于采样步长的分辨率。

再估计$E(d_{min})$.当$d_{min} < l$时模型不能良好拟合速度场，当$d_{min} < \Delta$时采样点可能落入OOD.

![[raw/assets/chaTrainingFreeRefinementFlow2026plus-1785241803614.webp]]

$l \ll \Delta \ll E(d_{min})$

$l \approx \Delta / 100$
$E(d_{min}) \approx 10-40 \times \Delta$

结论：问题不在于"模型能否拟合奇异点附近的速度场"（能），而在"离散化 step 是否恰好跨过奇异点导致 OOD"。低概率但一旦命中后果严重。

直观的解决方法有两个，如图所示：

1. 检测到v变化，缩小步长，直到v变化缩小
	![[chaTrainingFreeRefinementFlow2026plus-1785241855762.webp|209|170x227]]
2. 检测到v变化，增大步长，直到v变化缩小
	![[chaTrainingFreeRefinementFlow2026plus-1785241874885.webp|170]]


