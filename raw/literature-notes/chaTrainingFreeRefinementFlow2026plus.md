如[[zhangHIERARCHICALRECTIFIEDFLOW2025plus]]所言，我们的分位相交线段很少。三相交应该更少才对。故只考虑双相交线段。我们称相交出为奇异点，或者多模态点。

flowdpm


直线最小的距离为$d(t)_{min}$,离散步长为$\Delta = v\delta t$,模型的表达能力为$l$,代表模型不能良好拟合超过此尺度的分离路线：$d_{min}<l$时，v会在该尺度下被平均![[chaTrainingFreeRefinementFlow2026plus-1785231261637.webp]]


可以发现，交叉使得速度场发生剧烈的变化。这会导致一个后果：

1. 采样点落在这个剧烈的变化中心，导致ood

意味着$d(t)_{min}<\Delta$。

我目前猜测$\Delta \gg l$ ,$d(t)_{min}$的均值约为$\Delta$的10-40倍。但是我确实不能猜出$l$的具体范围。


ood的过程如图：![[raw/assets/chaTrainingFreeRefinementFlow2026plus-1785241803614.webp]]



直观的解决方法有两个，如图所示：

1. 检测到v变化，缩小步长，直到v变化缩小
	![[chaTrainingFreeRefinementFlow2026plus-1785241855762.webp|209|170x227]]
2. 检测到v变化，增大步长，直到v变化缩小
	![[chaTrainingFreeRefinementFlow2026plus-1785241874885.webp|170]]


