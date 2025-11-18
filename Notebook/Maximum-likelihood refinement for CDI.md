# Maximum-likelihood refinement for CDI

(Written before the birth of PIE)

```
Conjugate gradients
Preconditioning strategies
Regularization and typical modifications of the statistical noise model
```

## Prerequisition: Difference Map

Just skip it, the most important part is refinement.

## The main difference between photon and electron

1. Photon CDI/x-ray/optical

   Weak scattering

   Single interaction (Born /projection approximation)

   Usually no multiple scattering

   Probe is often defocused or Fresnel regime

   Object model: $O(x)= exp [i\phi(x)]$

2. Electron ptychography 

   Extremely strong scattering

   Multiple scattering $\rightarrow$ Multislice required

   Probe is formed by electron-optical lenses (high coherence)

   Object is atomic potential: $O(x,y)=exp[i\sigma V(x,z)]$

   So:

   The likelihood models

   Gradients

   Optimisation strategy

3. So this paragraph can be reused mathematically, but the forward model (probe-sample interaction) cannot be reused.

## PMF

### Electron Poisson PMF:

In the best scenario, the only source of noise is the counting statistics, giving a Poisson distribution.

$p(n_{jq}|P_r,O_r)=\frac{(I_{jq})^{n_{jq}}}{n_{jq}!}e^{-I_{jq}}$

r: real-space coordinate

j: index labeling the probe position

q: index leabeling detector pixel

$I_{jq}$: Intensity at the position q in the far-field plane in the absence of noise and other experimental errors./ Predicted intensity at pixel q for scan position j.

$n_{jq}$: Measured electron count at detector pixel q for scan position j.



### Standard Poisson PMF:

$P(k|\lambda)=\frac{\lambda^{k}}{k!}e^{-\lambda}$

Random counted variable k

Mean value $\lambda$



| Standard Poisson | Meaning             | Thibault notation | Meaning                                    |
| ---------------- | ------------------- | ----------------- | ------------------------------------------ |
| (k)              | observed count      | (n_{jq})          | measured photon count at position j,q      |
| (\lambda)        | expected mean count | (I_{jq})          | predicted intensity from the forward model |



The wave exiting the object has complex amplitude:    $\psi(x)$

The detector measures:
$$
I(q) = |\tilde{\psi}(q)|^2
$$
But the detector does **not** measure voltage or brightness — it counts **how many photons arrive**.

> More intensity → more photons expected
>  Less intensity → fewer photons expected

So:
$$
\lambda = I_{jq}
$$

## Why evolve into -log format?

### Minimize the **negative** log-likelihood:

$$
L = - \log \mathcal{L}
$$

Thus:
$$
L = -\sum_j \sum_q 
\left[n_{jq}\log I_{jq} - I_{jq} - \log(n_{jq}!)\right]
$$
This is exactly **Thibault’s negative log-likelihood (Eq. 4)**:
$$
L = - \sum_j \sum_q w_{jq} \left[n_{jq}\log I_{jq} - I_{jq} - \log(n_{jq}!)\right]
$$
Mask $w_{jq}$ is equal to 1 for valid pixels and to 0 for unmeasured regions, as well as bad or dead pixels.（屏蔽掉无效区域）

真实 detector：

- 经常会有坏像素（dead pixel）
- 边缘区有未测量像素
- 中心 beamstop 区域没有数据
- detector 会有 invalid region / saturation region



Thibault defines the gradients of the log-likelihood as:
$$
g_{O_r} = \sum_j P_{r-r_j} \chi_{jr}^*
$$

$$
g_{P_r} = \sum_j O_{r+r_j} \chi_{jr+r_j}^*
$$



This is Eq. (6) and (7). These gradients depend on **χ**.

==Wigner distribution function is totally different with wirtinger derivative.==

This step used wirtinger derivative. It greatly reduced calculation comlexity.

Only need the function is composed by z and z^alstra. 

------

Thibault's negative log-likelihood:
$$
L = -\sum_{j,q} w_{jq}[\,n_{jq}\log I_{jq} - I_{jq}\,]
$$
We want:
$$
\frac{\partial L}{\partial O_r^*}
$$
Chain rule:
$$
\frac{\partial L}{\partial O_r^*}
= \sum_{j,q}
\frac{\partial L}{\partial I_{jq}}
\frac{\partial I_{jq}}{\partial \tilde\psi_{jq}^*}
\frac{\partial \tilde\psi_{jq}}{\partial O_r^*}
$$
$I=∣\tilde ψ∣^2=\tildeψψ~*$

According to Wirtinger calculus:
$$
\frac{\partial I}{\partial \tilde\psi^*} = \tilde\psi
$$


Thibault defines:
$$
\tilde\chi_{jq}
= \frac{\partial L}{\partial I_{jq}} \, \tilde\psi_{jq}
= w_{jq}\left(1 - \frac{n_{jq}}{I_{jq}}\right) \tilde\psi_{jq}
$$
This IS the term:
$$
\frac{\partial L}{\partial I} \frac{\partial I}{\partial \psi^*}
$$
Thus the chain rule collapses to:
$$
\frac{\partial L}{\partial \tilde\psi^*} = \tilde\chi
$$
然后把 χ 作 inverse FFT 回 real space：
$$
\chi_{jr}=\mathcal{F}^{-1}(\tilde\chi_{jq})
$$


######  **relate the exit wave to object and probe**

Single-slice model:
$$
\psi_j(r) = P(r-r_j)\, O(r)
$$
Thus:
$$
\tilde\psi_{jq}
= \mathcal{F}\{ P(r-r_j) O(r)\}
$$
To get derivative w.r.t. the object:
$$
\frac{\partial \psi_j(r)}{\partial O(r')^\*}
= 0
$$
Because ψ depends on $O$, **not** on $O^\*$.

But using Wirtinger calculus, gradients are expressed w.r.t. **complex conjugate variable**:
$$
\frac{\partial L}{\partial O_r^\*}
= \sum_j P_{r-r_j}^\* \chi_{jr}
$$
This comes from applying the chain rule:
$$
\frac{\partial \psi_j(r)}{\partial O_r^\*} = 0, \quad
\frac{\partial \psi_j^\*(r)}{\partial O_r^\*} = P_{r-r_j}^\*
$$
Because:
$$
\psi_j^\*(r) = P^\*(r-r_j) O^\*(r)
$$
Wirtinger makes this **correct direction** automatic.

==tilde means Fourier domain==



------

So $\chi$ is a “bridge” between detector space (fourier domain) and object space (real domain)

## The definition of χ (Eq. 7)

$$
\tilde{\chi}_{jq}
= \frac{\partial L}{\partial I_{jq}}\,\tilde{\psi}_{jq}
= w_{jq}\left(1 - \frac{n_{jq}}{I_{jq}}\right)\tilde{\psi}_{jq}
$$

This is χ in **Fourier space**. (The tilde means Fourier domain.)

Then χ in **real space** is obtained by inverse Fourier transform:
$$
\chi_{jr} = \mathcal{F}^{-1}\{\tilde{\chi}_{jq}\}
$$
$\chi$ is defined as the error wave created by the chain rule.

在 Thibault 的 ML ptychography（最大似然）里，error 不是差值，而是：
$$
\chi_{jq}=\frac{\partial L}{\partial I_{jq}}\,\tilde{\psi}_{jq}= w_{jq}\left(1 - \frac{n_{jq}}{I_{jq}}\right)\tilde{\psi}_{jq}
$$
（L=-log likelihood (convex)）

数学上：

> “误差 = 如果你动一点 I，L 会增加多少？”

也就是：
$$
\frac{\partial L}{\partial I} =
1 - \frac{n}{I}
$$


解释：

- 当模型预测的 I 与真实计数 n 完全匹配：
  $$
  I = n \Rightarrow \frac{n}{I} = 1 \Rightarrow \frac{\partial L}{\partial I}=0
  $$
  → **没有误差**

- 当模型预测太高：
  $$
  I > n \Rightarrow \frac{n}{I} < 1\Rightarrow \partial L/\partial I>0
  $$
  → 往下修正

- 当模型预测太低：
  $$
  I < n \Rightarrow \frac{n}{I}>1\Rightarrow \partial L/\partial I<0
  $$
  → 往上修正

  也就是说：

  > **误差的方向 = gradient direction**
  >  **误差的大小 = gradient magnitude**

  这就是为什么 error = gradient 而不是 difference。

**那么 error=gradient 是巧合吗？**

不是巧合。

而是：

> **ML 的梯度 = 误差反投影（backprojection）**

更准确地说：

- **χ 是 detector plane 上的“残差波场”**
- 再乘以 probe/object 的点积 → 得到 real-space 的梯度

这不是 Empirical 的，是数学强制性的。

**为什么你觉得“好像很巧”？因为以下三点重合了：**

1. **error 形式来自噪声模型**（Poisson / Gaussian）
2. **ψ̃ 突然出现来自链式法则**
3. **反傅里叶变换把它从 detector plane 扔回 sample plane**

于是结构刚好变成：
$$
\nabla O = \sum_j P^* \cdot \chi_j
$$
是不是刚好长得像 PIE？
 对，这就是 Thibault 的大贡献：

> **他证明 PIE 更新公式不是 heuristic，而是最大似然的梯度下降近似。**

### Gaussian approximation

当中高剂量的时候，这时候可以忽略负光子的问题，负数的区域接近0。

最自然的估计：
$$
I_{jq} \approx n_{jq}
$$
所以：
$$
\sigma_{jq}^2 \approx I_{jq} \approx n_{jq}
$$

- 探测器记录的是 Photon counts（离散）

- 计数噪声本质上就是 **shot noise**

- shot noise 的标准差是：
  $$
  \sigma = \sqrt{N}
  $$

因此，高亮区（大 n）可靠、噪声低。
 低亮区（小 n）不可靠、噪声高。

把 σ² = njq 代入 Gaussian cost，就自动实现：

- 亮区域权重更大
- 暗区域权重变小
- 这完全符合 ptychography 的物理统计特性

因为 Poisson 在大 n 时逼近 Gaussian。

如果你有小于 5 的 photon count，作者建议用 Appendix A 的  **Bayesian prior**  来避免除零问题：
$$
\sigma^2 = n+1
$$

### Gaussian 模型更容易形成一个 **二次型 cost function**

在 ML-Ptycho 优化中，可以形成（8）式：
$$
L = \sum \frac{(I_{jq}-n_{jq})^2}{2\sigma_{jq}^2}
$$
这让：

- 梯度更简单
- Hessian 更简单
- 共轭梯度更稳定
- 线性化更自然
- 整体优化变成类似 “least-squares problem”

这就是为什么 ML reconstruction 会比 PIE/ePIE 收敛更平滑。





Thibault 2012 揭示了一个深刻联系 —— **Euclidean metric = Poisson ML 的二阶 Taylor 展开**

Euclidean metric = 振幅差的平方和
$$
\|\,|\psi| - \sqrt{n}\,\|^2
$$
是所有 classical phase retrieval algorithm 的默认 cost function

(ER, HIO, RAAR, PIE, ePIE, DM)

之所以著名，是因为它几十年来是 *唯一* 被广泛使用的 metric

Thibault 2012 证明：Euclidean metric 就是 Poisson likelihood 的二阶近似

**→ 所以 Euclidean metric 在 Poisson 噪声情况下是非常合理/物理正确的。**

这种二次型函数非常好优化，是最简单的函数，而且x>0单增，是凸函数。所以很被偏爱。而且hessian是常数，很简单。



### 梯度下降算法

## 0. 先对比一下：普通梯度下降在干嘛？

普通梯度下降：

- 当前点：$x^{(n)}$

- 梯度：$g^{(n)} = \nabla L(x^{(n)})$

- 更新方向：
  $$
  d^{(n)} = -g^{(n)}\quad\text{（最陡下降方向）}
  $$

- 再在这一条直线上找最优步长 $\gamma$：
  $$
  x^{(n+1)} = x^{(n)} + \gamma^{(n)} d^{(n)}
  $$

问题：
 如果等高线是“拉长的椭圆”（条件数很大），你会**来回“之字形”地抖动**，收敛很慢。

------

## 1. CG 的核心思想（一句话版）

> **CG 想做的是：在不显式算 Hessian 的前提下，让每一步的方向“在二次函数意义下相互独立（共轭）”，从而模拟牛顿法的效果，比最陡下降快很多。**

这里的关键字就是：**“共轭”**（conjugate）。

------

## 2. 什么叫“共轭方向”？

假设损失近似是一个二次型：
$$
L(x) = \frac12 x^\top A x - b^\top x + c
$$
其中 $A$ 是对称正定矩阵（可以理解为 Hessian）。

- **最陡下降**：每次方向都跟梯度走：
  $$
  d^{(n)} = -g^{(n)} = -\nabla L(x^{(n)})
  $$

- **共轭梯度**：希望构造一组方向 $\{d^{(0)}, d^{(1)}, \dots\}$，满足
  $$
  d^{(i)\top} A d^{(j)} = 0\quad (i \neq j)
  $$

这就叫 **A-共轭**（with respect to A）。直观感觉：

- 对二次函数来说，每次沿一个共轭方向做**一维最优**之后，
   以后再也不会“破坏”之前方向上已经优化好的结果。

所以理论上：

> 对 N 维二次函数，CG **最多 N 步就能到精确最优解**。
>  （而最陡下降可能要磨很多很多步）

当然在我们这种非凸、噪声 ML 里没有这么完美，但思路是从这里来的。

------

## 3. 真正的 CG 更新长什么样？

Thibault 论文里给的就是标准“非线性 CG”形式：

### 3.1 梯度

在第 $n$ 次迭代，先算梯度：
$$
g^{(n)} = 
\begin{pmatrix}
g^{(n)}_P \\
g^{(n)}_O
\end{pmatrix}
=
\begin{pmatrix}
\sum_j O^{(n)}_{r+r_j}\,\chi^{(n)*}_{jr} \\
\sum_j P^{(n)}_{r-r_j}\,\chi^{(n)*}_{jr}
\end{pmatrix}
\tag{13}
$$
这就是你看到的 (13) 式，针对 probe 和 object 两个块。

### 3.2 搜索方向

然后并不是简单地 $d^{(n)} = -g^{(n)}$，而是：
$$
d^{(n)} = -g^{(n)} + \beta^{(n)} d^{(n-1)}
\tag{14}
$$
可以类比为：

> “当前方向 = 负梯度 + 一点点上一次方向的残留”

其中 $\beta^{(n)}$ 用 Polak–Ribière 公式算：
$$
\beta^{(n)} = 
\frac{\langle g^{(n)}, g^{(n)} \rangle - \langle g^{(n)}, g^{(n-1)} \rangle}
     {\langle g^{(n-1)}, g^{(n-1)} \rangle}
\tag{15}
$$
这里的内积（scalar product）是：
$$
\langle g, h\rangle = g_P^\dagger h_P + g_O^\dagger h_O
\tag{16}
$$
也就是说：

- 把 probe 部分的梯度当成一个大向量 $g_P$
- 把 object 部分的梯度当成一个大向量 $g_O$
- 两边都做 Hermitian 内积再加起来

有点像把 $P, O$ 拼成一个巨大变量，然后在这个大空间里做共轭梯度。

> 🔑 你可以把 $\beta^{(n)}$ 理解为：“上一轮方向保留多少”的系数，它是自适应算出来的，而不是手动设的 momentum。

### 3.3 线搜索：沿着这个方向走多远？

方向有了，还要决定步长 $\gamma^{(n)}$：
$$
(P^{(n+1)}, O^{(n+1)}) = (P^{(n)}, O^{(n)}) + \gamma^{(n)} d^{(n)}
$$
Thibault 对 Gaussian 模型时指出：沿着这条线的 $L(\gamma)$ 是一个 8 次多项式，可以显式写出系数 $c_i$ 再求最小根：
$$
L(P+\gamma d_P, O+\gamma d_O) = \sum_{i=0}^{8} c_i \gamma^i
\tag{17}
$$
不过他们也说可以只近似到二次项，相当于做一次 Newton–Raphson 的线搜索，节省计算量。

对于 Poisson / Euclidean metric 的情形，就用几步 Newton–Raphson 迭代做 1D 最优化。

------

## 4. 和你熟悉的东西对照一下

### 4.1 和普通 gradient descent 比

- GD：
  $$
  d^{(n)}_{\text{GD}} = -g^{(n)}
  $$

- CG：
  $$
  d^{(n)}_{\text{CG}} = -g^{(n)} + \beta^{(n)}d^{(n-1)}
  $$

相当于：

> 在“负梯度”基础上，加上一点“沿着过去方向的滑行”，但这不是简单 momentum，而是经过 Polak–Ribière 精调，使得方向之间趋向共轭。

### 4.2 和动量 / Adam 比

- Momentum：
  $$
  v^{(n)} = \mu v^{(n-1)} - \eta g^{(n)},\quad x^{(n+1)} = x^{(n)} + v^{(n)}
  $$

这里 $\mu$ 是手调参数。

- CG：
  $$
  d^{(n)} = -g^{(n)} + \beta^{(n)} d^{(n-1)}
  $$

$\beta^{(n)}$ 是由两次梯度自动算出来的，和局部曲率结构有关——这就是它“伪二阶”的来源。

### 1. 什么是“最陡梯度方向”？

我们有一个损失函数 $L(x)$，它是一个高维函数（比如 ptychography 的 negative log-likelihood）。
 最陡梯度方向指的是：

> **从当前位置往哪个方向走，能让 L 下降得最快？**

这个方向就是：
$$
-\nabla L(x)
$$
为什么是负梯度？因为**梯度（∇L）是上升最快的方向**，所以负梯度是下降最快的方向。

------

###  2. 那么“怎么知道哪个方向是最陡的”？

关键数学结论：

> 在所有单位方向 $d$ 中，使得 $L(x+\epsilon d)$ 降得最快的方向就是 $d = -\nabla L$。

这个结论**不需要 Hessian**，只需要一阶导数（梯度）就足够了。

### 证明直觉（非常简单）：

我们看 $L$ 在点 $x$ 附近一阶泰勒展开：
$$
L(x+\epsilon d) \approx L(x) + \epsilon\, d^\top \nabla L(x)
$$
下降最快，就是让 $d^\top \nabla L(x)$ 尽可能负。

因为 $||d||=1$，当：
$$
d = -\frac{\nabla L}{\|\nabla L\|}
$$
时它最负。

于是：
$$
\text{最陡下降方向} = -\nabla L(x)
$$
**所以“最陡”不需要 Hessian，完全可以只靠梯度决定。**

------

### 3. 那 Hessian 是干嘛的？

Hessian（二阶导数）告诉你：

- 这个函数的“弯曲程度”
- 每一个方向的“曲率”

如果你要找到**最优步长 γ**（line search），Hessian 能告诉你“弯得有多厉害”。

对于简单的二次函数：
$$
L(x)=\frac12 x^T A x - b^T x
$$
最优方向其实是：
$$
p = -A^{-1}\nabla L
$$
它比负梯度聪明得多，因为它已经知道曲率。

但是！！！

- ptychography 的维度巨大（几十万变量）
- Hessian 是几十万 × 几十万 的矩阵
- 根本存不下，更算不出来

所以不能用 Newton 法（需要 Hessian）。

------

###  4. CG（共轭梯度）为什么牛？

CG 的作用就是：

> **不用显式 Hessian，也能实现“像 Newton 一样的聪明方向”。**

它通过构造“共轭方向”来模拟 Hessian 的信息。

你现在知道：

### - 最陡方向 = 负梯度

### - Newton 方向 = -H^{-1}g （太贵）

### - CG 方向 = -g + β d_prev （既不贵又很强）

β 的作用是：

- 保留前一轮的方向信息
- 自动调节，模拟 Hessian 曲率
- 让方向之间“互不干扰”（共轭）

这就是为什么 CG 能在高维空间表现极好。

------

###  5. 一句话彻底总结三者：

### ✔ 最陡下降（steepest descent）

方向 = -梯度
 → 不用 Hessian
 → 最笨、最慢

------

### ✔ Newton 法

方向 = -H^{-1}g
 → 最聪明
 → 需要 Hessian（不可能用）

------

### ✔ Conjugate Gradient（CG）

方向 = -g + β d_prev
 → 不用 Hessian
 → 方向“几乎像 Newton 一样聪明”
 → 所以在 ptychography ML 中能跑得很快（论文指定方法）



