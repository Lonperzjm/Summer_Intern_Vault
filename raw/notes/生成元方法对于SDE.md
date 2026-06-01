生成元法主要用来做一件事：

$\boxed{ \text{把一个随机过程的“局部运动规则”转成可以推导分布、条件过程、反向过程的微分方程工具。} }$
这里总结的是 Doob's $h$-transform 这条公式，在论文[[raw/literature-notes/zhouDenoisingDiffusionBridge2023]],生成元方法在  “4. 生成元方法” 章节

  

$$

dX_t

=

f(X_t,t)\,dt

+

g(t)^2 \nabla_x \log p(X_T=y\mid X_t)\,dt

+

g(t)\,dW_t,

\qquad

X_T=y.

$$

## 1. 这个公式的意义

  

它的意义是：

  

$$

\boxed{

\text{把一个普通扩散过程，变成“条件于终点 } X_T = y\text{”的扩散过程。}

}

$$

  

它表示：在原始扩散的基础上，额外加了一个“终点吸引漂移”：

  

$$

g(t)^2 \nabla_x \log p(X_T = y \mid X_t).

$$

  

这个项的方向是：**让当前位置稍微移动后，未来到达 $y$ 的概率密度增加最快的方向**。

  

## 2. 这个公式怎么来的：第一步，定义 $h$

  

令

  

$$

h(x,t)

=

p(X_T=y\mid X_t=x).

$$

  

它表示：当前在 $x$，未来到达 $y$ 的可能性。

  

## 3. 第二步，用贝叶斯公式得到条件转移概率

原始过程的转移密度记作：

  

$$

p_{s,t}(x,z)

=

p(X_t=z\mid X_s=x).

$$

现在考虑条件过程：

  

$$

p(X_t=z\mid X_s=x,X_T=y).

$$

由贝叶斯公式：

  

$$

p(X_t=z\mid X_s=x,X_T=y)

=

\frac{

p(X_t=z,X_T=y\mid X_s=x)

}{

p(X_T=y\mid X_s=x)

}.

$$

  

利用马尔可夫性：

  

$$

p(X_t=z,X_T=y\mid X_s=x)

=

p_{s,t}(x,z)\,p_{t,T}(z,y).

$$

  

所以：

  

$$

p(X_t=z\mid X_s=x,X_T=y)

=

p_{s,t}(x,z)

\frac{

p_{t,T}(z,y)

}{

p_{s,T}(x,y)

}.

$$

  

令：

  

$$

h(z,t)=p_{t,T}(z,y),

\qquad

h(x,s)=p_{s,T}(x,y),

$$

  

得到：

  

$$

p_{s,t}^h(x,z)

=

p_{s,t}(x,z)

\frac{

h(z,t)

}{

h(x,s)

}.

$$

  

## 4. 生成元方法

  
  

### 4.1 生成元的定义

给定 SDE：

  

$$

dX_t

=

f(X_t,t)\,dt

+

g(t)\,dW_t,

$$

  

生成元 $\mathcal{L}_t$ 是一个作用在测试函数 $\phi(x)$ 上的微分算子，定义为：

  

$$

\mathcal{L}_t\phi(x)

=

\lim_{\Delta t\to 0}

\frac{

\mathbb{E}\left[\phi(X_{t+\Delta t})\mid X_t=x\right]

-

\phi(x)

}{

\Delta t

}.

$$

  

意义是跟踪在 $t$ 时刻在 $x$ 的粒子，$\Delta t$ 后它的测试函数 $\phi(x)$ 的期望变化。注意到：

  

$$

\mathbb{E}

\left[

h(X_{t+\Delta t}, t+\Delta t)

\mid

X_t=x

\right] = h(x,t) + \Delta t

\left[

\partial_t h(x,t)

+

\mathcal{L}_t h(x,t)

\right] + o(\Delta t). = h(x,t)

$$

  
  

所以：

  

$$

\partial_t h(x,t)

+

\mathcal{L}_t h(x,t)

=

0.

$$

这个方程叫 Kolmogorov backward equation。

  

对 SDE

  

$$

dX_t

=

f(X_t,t)\,dt

+

g(t)\,dW_t,

$$

  

由 Itô 公式得到：

  

$$

\mathcal{L}_t\phi

=

f\cdot\nabla\phi

+

\frac{1}{2}g(t)^2\Delta\phi.

$$

  

### 4.2 生成元和密度演化的关系

  

如果 $X_t$ 的密度是 $p_t(x)$，那么对任意测试函数：

  

$$

\mathbb{E}[\phi(X_t)]

=

\int

\phi(x)p_t(x)\,dx.

$$

  

从“函数沿随机过程变化”的角度看：

  

$$

\frac{d}{dt}\mathbb{E}[\phi(X_t)]

=

\int

p_t(x)\,

\mathcal{L}_t\phi(x)\,dx.

$$

  

从“密度本身变化”的角度看：

  

$$

\frac{d}{dt}\mathbb{E}[\phi(X_t)]

=

\int

\phi(x)\,

\frac{\partial p_t(x)}{\partial t}

\,dx.

$$

  

为了让这两种看法一致，定义伴随算子 $\mathcal{L}_t^*$：

  

$$

\int

p_t(x)\,

\mathcal{L}_t\phi(x)\,dx

=

\int

\phi(x)\,

\mathcal{L}_t^*p_t(x)\,dx.

$$

  

于是：

  

$$

\frac{\partial p_t}{\partial t}

=

\mathcal{L}_t^*p_t.

$$

  

这就是 Fokker-Planck 方程。具体来说：

  

对

  

$$

\mathcal{L}_t\phi

=

f\cdot\nabla\phi

+

\frac{1}{2}g^2\Delta\phi,

$$

  

通过积分分部得到：

  

$$

\mathcal{L}_t^*p

=

-

\nabla\cdot(fp)

+

\frac{1}{2}g^2\Delta p.

$$

  

因此：

  

$$

\frac{\partial p_t}{\partial t}

=

-

\nabla\cdot(fp_t)

+

\frac{1}{2}g^2\Delta p_t.

$$

  

### 4.3 Doob’s $h$-transform 的生成元

  

Kolmogorov backward equation

  

$$

\partial_t h(x,t)

+

\mathcal{L}_t h(x,t)

=

0.

$$

  

原过程的转移算子是：

  

$$

P_{s,t}\phi(x)

=

\int

p_{s,t}(x,z)\phi(z)\,dz.

$$

  

条件化后的转移算子是：

  

$$

P_{s,t}^h\phi(x)

=

\int

p_{s,t}^h(x,z)\phi(z)\,dz.

$$

  

代入 $h$-transform：

  

$$

P_{s,t}^h\phi(x)

=

\frac{1}{h(x,s)}

\int

p_{s,t}(x,z)\,

h(z,t)\,

\phi(z)\,

dz.

$$

  

所以：

  

$$

P_{s,t}^h\phi(x)

=

\frac{1}{h(x,s)}

P_{s,t}(h\phi)(x).

$$

  

新生成元是新转移算子的瞬时变化率。结合

  

$$

\partial_t h+\mathcal{L}_t h=0,

$$

  

可以得到：

  

$$

\mathcal{L}_t^h\phi

=

\frac{1}{h}

\mathcal{L}_t(h\phi)

-

\frac{\phi}{h}

\mathcal{L}_t h.

$$

  

展开得到：

  

$$

\mathcal{L}_t^h\phi

=

\left[

f

+

g^2\nabla\log h

\right]\cdot\nabla\phi

+

\frac{1}{2}g^2\Delta\phi.

$$

  

## 5. 结论

由生成元的形式可以看出，$h$-transform 后的 SDE 是：

  

$$

dX_t

=

\left[

f(X_t,t)

+

g(t)^2\nabla_x\log h(X_t,t)

\right]dt

+

g(t)\,dW_t.

$$