假设干净变量 $x \sim p(x)$，观测到带高斯噪声的变量：

$$
y = x + \sigma \epsilon, \qquad \epsilon \sim \mathcal{N}(0, I).
$$

那么 $y$ 的边缘分布记为 $p_Y(y)$，它的 score 是：

$$
\nabla_y \log p_Y(y).
$$

Tweedie 公式说：

$$
\mathbb{E}[x \mid y] = y + \sigma^2 \nabla_y \log p_Y(y).
$$

。