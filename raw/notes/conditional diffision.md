要解决$p(y|x_0)$的问题，已知：$$\nabla_{y}\log(p(y|x_0))=\nabla_{y}\log(\frac{p(x_0|y)p(y)}{p(x_0)})=\nabla_{y}\log(p(y))+\nabla_{y}\log(p(x_0|y))$$
又假设$p(x_0|y)=\exp(-E(x_0,y,t))$ ，其中要求$\exp(-E(x_0,y,t))$满足归一化$Z(y,t)=\int \exp(-E(x_0,y,t))dx_0=1$ ，可得：$$\nabla_{y}\log(p(y|x_0))=s_0-\nabla_{y}E(y,x_0,t)$$
值得一提的是，如果是cat-dog这种任务，$p(x|y)$并不会比$p(y|x)$更加简化

更不牵强的做法是：
$$
\tilde{p}_t(y \mid x_0)
=
\frac{p_t^{\mathcal{Y}}(y)\exp[-\mathcal{E}(y,x_0,t)]}{Z_t(x_0)}.
$$

注意这里的归一化常数是：

$$
Z_t(x_0)
=
\int p_t^{\mathcal{Y}}(y)\exp[-\mathcal{E}(y,x_0,t)]\,dy.
$$

它是对 $y$ 积分得到的，但是作为函数，它依赖的是条件 $x_0$，不依赖当前变量 $y$。

所以对 $y$ 求梯度时：

$$
\nabla_y \log Z_t(x_0)=0.
$$

因此：

$$
\nabla_y \log \tilde{p}_t(y \mid x_0)
=
\nabla_y \log p_t^{\mathcal{Y}}(y)
-
\nabla_y \mathcal{E}(y,x_0,t).
$$

也就是：

$$
\nabla_y \log \tilde{p}_t(y \mid x_0)
=
s_{\mathcal{Y}}(y,t)
-
\nabla_y \mathcal{E}(y,x_0,t).
$$