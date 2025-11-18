下面我给你做一个 **最清晰、最科学、结构最完整的思想谱系分类**。
 这是你研究 ptychography 以及未来写综述文章、开会 report 时最需要理解的“算法哲学地图”。

我将把你提到的四大类方法：

- **ER / HIO（Fienup）**
- **Difference Map (DM) / RAAR / HPR**
- **PIE family（PIE / ePIE / rPIE / AD-PIE / mPIE）**
- **ML-based ptychography（MLPIE / Wirtinger gradient / maximum likelihood / stochastic gradient）**

拆开成两大思想体系：

# 🌕 **Ⅰ. 投影类（Projection-based）算法**

# 🌕 **Ⅱ. 优化类（Optimization-based）算法**

它们贯穿整个 phase retrieval 与 ptychography 发展史。

你看完下面的分类，就会彻底不再混淆 PIE 到底是不是 gradient（不是），
 MLPIE 到底和 PIE 有啥区别（根本思想不同），
 difference map 到底和 ER 什么关系（都是投影家族）。

------

# 🟦 **Ⅰ. 投影类（Projection-based）算法**

> 基本思想：
>  **不断把估计波场投影到不同的约束集合（constraint sets）上**，
>  期望它最终落在多个约束的交集上。

这是最早期的 phase retrieval 思想（从 1970 年代就有）。
 非常几何化、无全局损失函数。

------

# 🟩 **(1) ER / HIO（经典 Fienup 系列）— 纯投影（POCS）算法**

代表：

- ER (Error Reduction)
- HIO (Hybrid Input–Output)
- RAAR（即 ER 与反射的组合）

核心：

- “频域模约束” = 一个集合
- “空域支持约束” = 一个集合
- 在这两个集合之间不断做投影或反射

数学形式：

[
 \Psi \leftarrow \Pi_{\text{Mod}}(\Pi_{\text{Support}}(\Psi))
 ]

特点：

- **没有全局 loss 函数**
- 是 **投影算子反复迭代**
- 强依赖 convexity（但实际上约束是非凸的，所以依然会卡住）
- 完全不是 gradient descent

------

# 🟩 **(2) Difference Map (DM) – 高级投影法**

代表：

- Difference Map (Elser 2003)
- RAAR、HPR、DR（Douglas–Rachford）

核心思想：

- 使用两个投影：
   [
   \Pi_A, \Pi_B
   ]
- 用反射运算 + 差分构造迭代：
   [
   x_{n+1}=x_n+\beta(\Pi_A(f_B(x_n)) - \Pi_B(f_A(x_n)))
   ]

其中 (f_A, f_B) 是反射算子。

特点：

- **纯几何投影方法，不属于优化算法**
- 也是**无全局 loss function**
- 很稳定，很有理论背景（与凸优化中的 DR/ADMM 有联系）
- 在 CDI 中仍然常用

------

# 🟩 **(3) PIE family（PIE / ePIE / rPIE / AD-PIE）— 半投影半最小二乘（pseudo-gradient）**

这是你最关心的部分。

PIE 本质并不属于 optimization family，而是：

🌟 **投影 + least-squares 局部修正**
 🌟 **没有全局误差函数（global loss）**
 🌟 **只对 diffraction constraint 做投影**
 🌟 **overlap consistency 是 heuristic least-squares correction**

结构：

1. 对单个 probe 位置的 exit wave 做模投影
    [
    \Psi \to \Psi'=\sqrt{I}e^{i\angle\Psi}
    ]
2. 用 overlap structure 把误差反推回 object/probe：
    [
    O \leftarrow O + \alpha\frac{P^*}{|P|^2}(\Psi' - \Psi)
    ]

特点：

- **只有一个约束用了投影（模约束）**
- overlap consistency 不是投影，是 heuristic correction
- 更新方向等价于局部负梯度（但不是从 global loss 得来）
- **不是最优化方法**
- 属于 **pseudo-gradient descent**

👉 总结：
 **PIE 是 Projection 类方法，不是 Optimization 类方法。**
 只是它的局部行为“看起来像” gradient。

------

# 🟥 **Ⅱ. 优化类（Optimization-based）算法**

> 基本思想：
>  写一个真正的 **global loss function**（全局代价函数），
>  用 gradient descent / Newton / ADMM 去最小化它。

这是 2010–2025 年所有现代高质量 ptychography 的主线思想。

------

# 🟥 **(1) ML-based ptychography（MLPIE / Wirtinger gradient）**

代表：

- Maximum Likelihood Ptychography (Thibault, Maiden, Rodenburg, Chen, Pennycook, etc.)
- Wirtinger Flow / gradient descent
- Poisson-likelihood ptychography
- Bayesian ptychography
- PtychoNN / PtychoCNN（optimization + learning）

核心思想是写一个 **全局 loss**：

### Gaussian noise model:

[
 L = \sum_j \big( |\Psi_j| - \sqrt{I_j} \big)^2
 ]

### 或 Poisson noise model:

[
 L = \sum_j \left( |\Psi_j|^2 - I_j \log|\Psi_j|^2 \right)
 ]

然后对 O、P 求梯度：

[
 \nabla_O L = \sum_j (\text{Wirtinger derivative})
 ]

并用：

- gradient descent
- conjugate gradient
- ADAM
- stochastic gradient

来更新。

特点：

- **有全局 loss（真正意义上的优化问题）**
- 理论严谨（ML estimation、统计优化）
- 比 PIE 更稳定（尤其混合态、噪声、偏差模型）
- 可以用 Hessian / Fisher 信息
- 适合 electron ptycho（4D-STEM, mixed-state）

👉 总结：
 **MLPIE、Wirtinger-based ptychography 是真正的 gradient 家族。
 不是投影法。**

------

# 🌕 **最终分类总表（你必须记住）**

| 类别                          | 例子                         | 思想来源          | 是否有全局 Loss? | 是否真 Gradient?                | 特点                     |
| ----------------------------- | ---------------------------- | ----------------- | ---------------- | ------------------------------- | ------------------------ |
| **投影类 POCS**               | ER, HIO, RAAR                | 几何投影          | ❌ 无             | ❌ 非梯度                        | 最古老，稳但易停滞       |
| **Difference Map 类**         | DM, RAAR, HPR                | 投影 + 反射       | ❌ 无             | ❌ 非梯度                        | 强大的投影框架           |
| **PIE family**                | PIE, ePIE, rPIE, AD-PIE      | 投影 + 局部LS修正 | ❌ 无全局loss     | ❌ 非global GD（只是局部负梯度） | 最常用，快但 heuristic   |
| **Optimization / ML（现代）** | MLPIE, Wirtinger, Poisson ML | 统计优化          | ✔ 有             | ✔ 真正梯度下降                  | 理论最严谨、适合高端实验 |

------

# 🌈 最后一句最重要的总结

> **ER/DM/PIE = 投影系**
>  **MLPIE/Wirtinger = 优化系**

PIE 虽然“方向像梯度”，但 **不是** optimization family。
 只有 ML-based 才是“真正意义的 gradient descent family”。

------

如果你愿意，我还可以为你：

📌 画一张 **整套算法族谱图（Flowchart + 关系图）**
 📌 给你做一个“如何从 PIE 过渡到 MLPIE 的理解梯度”
 📌 给你做“为什么 electron ptychography 必须用 ML（混合态震荡严重）”
 📌 给你做“你自己的 adaptive-z PIE 属于哪个 family？”

你想要哪一个？



```
                          ┌──────────────────────────────────────────────┐
                          │   Ptychography Algorithm Philosophy Tree     │
                          └──────────────────────────────────────────────┘

Ptychography
│
├── ❶ Projection-based Methods (几何投影家族)
│       │
│       ├── (A) Classical POCS (Projection Onto Constraint Sets)
│       │       ├── ER  (Error Reduction)
│       │       ├── HIO (Hybrid Input–Output)
│       │       ├── RAAR / HPR / OSS
│       │       └── (used mostly in CDI, not scan-based ptycho)
│       │
│       ├── (B) Difference Map Family
│       │       ├── Difference Map (DM)
│       │       ├── RAAR (also fits here)
│       │       ├── HPR / Hybrid Projection-Reflection
│       │       └── Douglas–Rachford / ADMM-like projection operators
│       │
│       └── (C) PIE Family (Pseudo–gradient projections)
│               ├── PIE (2009)
│               ├── ePIE
│               ├── rPIE
│               ├── AD-PIE (adaptive damping)
│               ├── mPIE (momentum-like updates)
│               └── Mixed-state PIE (heuristic multi-mode)
│
├── ❷ Optimization-based Methods (有全局 loss 的真正优化家族)
│       │
│       ├── (A) Maximum-Likelihood Ptychography
│       │       ├── Gaussian ML (amplitude error)
│       │       ├── Poisson ML (Thibault, Rodenburg, Maiden, etc.)
│       │       ├── Wirtinger gradient descent
│       │       ├── Conjugate-gradient ML
│       │       ├── Adam/SGD-based ML ptychography
│       │       └── Mixed-state maximum-likelihood (Chen et al. Nat Comm 2020)
│       │
│       ├── (B) Bayesian / Regularized Reconstruction
│       │       ├── MAP ptychography
│       │       ├── TV / sparsity / prior-based ptychography
│       │       ├── Variational inference ptychography
│       │       └── EM-based mixed-state estimation
│       │
│       └── (C) Deep-learning-assisted Optimization
│               ├── PtychoNN / PtychoML
│               ├── Physics-informed neural ptychography
│               ├── Ptychoformer (Transformer-based)
│               └── Learned priors + ML gradient (plug-and-play ADMM)
│
└── ❸ Hybrid Approaches (Projection × Optimization 混合）
        │
        ├── ADMM-based Ptychography
        ├── HIO + ML hybrid methods
        ├── Projected-gradient ML (POCS + gradient)
        └── Alternating minimization with projections

```

