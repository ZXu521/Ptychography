Probe damping effect

在电子系统中，**probe** 指的是入射电子波函数 $\Psi_{\text{probe}}(\mathbf{r})$，通常由收敛电子束形成（例如STEM探针）。
 在理想情况下，这个波前的强度和相位在传播到样品表面前是稳定的。
 但在真实系统中，会出现多种“阻尼效应”（damping effects），使得这个探针不再完美。

## 整体发展脉络（2004 → 2025）

| 阶段                           | 代表算法                                                     | 主要创新                                                     | 数学本质                    |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------- |
| **① 初代迭代引擎 (2004–2009)** | **PIE** (2004–2009) Maiden & Rodenburg 2009, *Ultramicroscopy* | 提出扫描重叠+误差反演思想；单探针、单物体、强相位近似        | 投影到约束集（POCS）类算法  |
| **② 改进型稳定化 (2009–2014)** | **ePIE** (2009) **rPIE** (2015, Maiden et al.)               | ePIE: 同步更新 probe 与 object； rPIE: 比例调节增益、稳定高动态范围 | 梯度下降近似（ADMM-like）   |
| **③ 动量与鲁棒性 (2016–2018)** | **mPIE / SR-PIE / momentum-PIE**                             | 引入动量项、归一化能量、子像素插值等，提升收敛               | 一阶优化的数值加速          |
| **④ 统计建模阶段 (2018–2020)** | **Maximum-Likelihood Ptychography** (Odstrčil 2018, *Opt. Express*) | 从经验误差→明确似然函数（Poisson噪声模型），用梯度下降求解   | 最大似然估计 (MLE) 优化问题 |
| **⑤ 混态模型阶段 (2020– )**    | **Mixed-State ML-Ptychography** (Chen et al. 2020, *Nat. Commun.*) | 建模探针部分相干性；多模态 probe；mini-batch 优化；GPU 加速  | 概率混态 + MLE              |
| **⑥ 学习驱动阶段 (2021–2025)** | **Neural-PIE / Deep Ptycho / PtychoNN / Ptychoformer**       | 用深度神经网络学习更新映射、先验正则化、物体统计模型         | 深度学习 + 变分贝叶斯优化   |

DPC 方法的优势是**线性、快速**；缺点是只恢复梯度，不完整。
 而 PIE 的优势是**非线性恢复能力强**，但计算量大。

于是，有研究者提出结合两者：
 → 用 DPC 的梯度信息作为先验或误差信号，引导 PIE 的迭代更新。
 这就是 **Differential-Contrast PIE（DC-PIE）**。

DC-PIE 的迭代核心仍然沿用 ePIE 框架，只是修改了更新误差项：

传统 PIE 使用：
$$
\Delta \Psi = \Psi_{\text{meas}} - \Psi_{\text{calc}}
$$
而 DC-PIE 使用：
$$
\Delta \Psi_{\text{DC}} = \nabla \Psi_{\text{meas}} - \nabla \Psi_{\text{calc}}
$$
之后就是maximum-likelihood & bayesian & Variational gradiant ptychography

## 两条路径的核心哲学

| 方向                                                         | 核心思想                                                     | 优点                             | 局限                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------- | ------------------------------------------ |
| **统计学流派**   (*Maximum Likelihood / Bayesian / Variational PIE*) | 从物理 forward model 出发，以概率模型构建 loss function，如   $\mathcal{L}(O,P)=\sum(I_{calc}-I_{meas}\ln I_{calc})$ | 物理一致性强、可解释、数据需求低 | 计算量大、先验假设有限、难以捕捉复杂非线性 |
| **深度学习流派**   (*Deep / Neural Ptycho / Ptychoformer*)   | 用神经网络近似重建映射或正则项；例如 U-Net/Transformer 直接预测相位 | 重建速度快，可学习复杂先验       | 缺乏物理约束，外推性差，容易“幻觉”结构     |

##  当代PIE家族研究的三大主线

| 路线                                             | 代表方向                                                     | 主要目标                       | 特征                           |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------ | ------------------------------ |
| **A. 物理模型派（Physical Modeling Line）**      | zPIE, multi-slice PIE, Fresnel / near-field, partial coherence modeling | 更准确的波前传播与样品厚度建模 | 基于波动方程，适合实验物理研究 |
| **B. 统计优化派（Statistical / Bayesian Line）** | ML-PIE, mixed-state, variational inference                   | 用概率方法建模噪声、相干性     | 数学优化为核心                 |
| **C. 深度学习派（Neural Line）**                 | Neural-PIE, Ptychoformer, Deep Bayesian PIE                  | 数据驱动加速重建与先验学习     | 工程化实现快，物理解释弱       |



A、B、C可以hybrid。



| **Neural-PIE / PtychoNN (2021–)** | 用神经网络近似迭代映射，加速收敛 |
| --------------------------------- | -------------------------------- |
|                                   |                                  |

| **Deep Ptychography (2022–)** | 将物体、probe 作为隐变量，通过深度生成网络正则化 |
| ----------------------------- | ------------------------------------------------ |
|                               |                                                  |

| **Ptychoformer (2023)** | Transformer 模型直接从 4D 数据预测相位（弱监督） |
| ----------------------- | ------------------------------------------------ |
|                         |                                                  |

| **Bayesian Ptychography (2024)** | 结合 MLE 与先验，形成变分贝叶斯重建框架 |
| -------------------------------- | --------------------------------------- |
|                                  |                                         |

![Deep Bayesian Pty](./Chen%20Zhen%20paper%20study.assets/Deep%20Bayesian%20Pty.png)

其中：

- 第一个项是物理 forward model；
- 第二个项由神经网络（或扩散模型）实现 learned regularizer。

这就是 **“Deep Bayesian PIE”** 的核心公式 ——
 目前在 PSI (Odstrčil)、Cambridge (Nellist)、Berkeley 等小组都有相关研究。

------

## setup

SOURCE

OBJECT LENS

SAMPLE （薄、低散射）

DETECTOR

$$
N_F = \frac{D^2}{\lambda z}
$$
$\lambda:波长$

$z: 探测器到样品距离$

$D: 探针照明区域直径$

波长越短，波前的曲率变化越快；

因此同样距离下，短波（电子）传播“更快趋于平面波”；

所以电子系统天生“容易远场”，而光学系统必须靠透镜做傅里叶变换。

但是由于电镜观察的样本需要更加小的照明区域，根据fesnel number的公式，实际上多数电镜又实际在fresnel区。因为 λ 很小，而 D 通常几微米，如果自然达到fraunhofer:
$$
z \text{ 需要几米甚至几十米！}
$$


但fresnel number的公式里z是detector到sample的距离啊。我不明白为什么物镜还可以让成像更容易进入fraunhofer区。

其实正是**波动光学里“透镜等效传播”的核心原理**

其实还是你理解的那样。物镜是sample和detector之间的。物镜的定义是最接近物体的lens。

但是引入object lens 会导致相位畸变。tem作为相关技术的产物已经被淘汰。


陈震的文章研究的sample是薄样品、且弱散射，2D material。

Mixed-state electron ptychography

并且电子束是部分相干的。



此外optical是水平移动光源，而electron只是改变入射角，造成一个tilt beam。

z与$\theta$有比例关系。

### problems:

#### Partial coherence of the probe

#### Focus

In-focus probe 和defocus probe主要是指焦点在不在样品平面上。

根据fresnel number公式，

in-focus 是最优选择，因为可以达到fraunhofer条件（fourier）而且最大化了信噪比和相干性（？），大大减少了计算量。

//coherence

但是有时依然会使用defocus/fresnel模型

| 情况                       | 原因                                                         | 替代方案                      |
| -------------------------- | ------------------------------------------------------------ | ----------------------------- |
| **厚样品 (≥ 50 nm)**       | 电子波在样品中多次散射 → 不能用单层透射近似                  | Multi-slice or adaptive-z PIE |
| **中低相干源 / 发射面大**  | 传播导致相干长度缩短 → in-focus 假设失效                     | Mixed-state PIE               |
| **需要深度信息或体层重建** | 单平面探针无法获取 z 信息                                    | z-PIE / 3D ptychography       |
| **光学或软 X-ray 波段**    | 波长较长、传播距离相对短 → 很难进入Fraunhofer区。只能Fresnel。 | Fresnel-PIE / z-PIE           |

主要是光学的overlap主要在specimen上，而electron即使不在specimen上overlap，还可以在detector上overlap.

electron 可以双重overlap。

##### coherence

在电镜或光学系统中，通常讲两种相干性：

| 类型                                | 含义                          | 典型影响                       |
| ----------------------------------- | ----------------------------- | ------------------------------ |
| **空间相干性** (spatial coherence)  | 波前上不同点之间的相位相关性  | 影响衍射对比度、干涉条纹清晰度 |
| **时间相干性** (temporal coherence) | 不同时间/能量成分的相位稳定性 | 影响能量展宽、chromatic blur   |

而在 ptychography 中，最关键的是**空间相干性**。
 因为我们希望照明波前在样品区域内**相位一致（有定义的复振幅）**，这样重叠区域的相位才能被算法利用。

##### 为什么“in-focus”探针相干性最好

要理解这一点，得从“波的传播过程”看相干性是如何演化的。

###### 传播会引入波前相位差

当波前从焦点传播到样品面（或离焦传播）时，不同空间点的传播路径长度不同。
 相干场可写作：
$$
\Psi(x,y,z) = P(x,y)\, e^{i k \frac{x^2 + y^2}{2R(z)}}
$$
其中 $R(z)$ 是波前曲率半径。

→ 离焦越远，曲率项 $e^{i k (x^2+y^2)/(2R)}$ 变化越快，
 波前相位在横向上不再“同步”。
 → 这会导致**空间相干性下降**。

------

######  聚焦在样品平面时 (in-focus)

此时 $R(z) → ∞$，波前基本平坦。
 即：
$$
\Psi_\text{focus}(x,y) \approx P(x,y)\, e^{i k z}
$$
整个照明区域内的相位几乎一致，
 → 各点的相对相位稳定、干涉对比度最高。

换句话说：

> In-focus = 波前相位最“平坦”的状态 = 相干长度在样品上最大化。

------

###### 离焦传播的相干退化

在 defocus 或 Fresnel 区传播时：

- 波前内部不同点的路径差 $Δr$ 可能超过相干长度 $L_c$；

- 各点之间的相位关系被“打散”，

- 探针成为部分相干的叠加：
  $$
  \rho_P = \sum_i w_i |P_i\rangle\langle P_i|
  $$

**因此在 defocus 条件下，即使初始源是高相干的，传播后也会变成多模混合态。**

Low dose 的方法：detector使用empad而不是ccd.获取diffraction pattern更快。

