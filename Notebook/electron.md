# Electron

## 总体分类框架

setup

数据模型/datasets

算法表格/对比图

（其他的如历史描述、应用举例可以省略）

此外光与electron作为incident的区别决定了：

该不该用投影近似；

传播公式是Frauhofer还是Fresnel;

哪些改进（比如动量、相干性建模）才是真的有意义

------

[TOC]



线性相位近似法（DPC） → 半线性解析法（SSB） → 非线性迭代优化法（PIE）

           -  ┌─────────────────────┐
           |  │  DPC-STEM           │  →  线性、单次扫描、只提取相位梯度
           |  └─────────────────────┘
    Direct method        ↓ interim： Center of Mass STEM （pty的最简线性版本）
           |  ┌─────────────────────┐
           |  │  SSB Ptychography   │  →  半线性、利用盘干涉推相位
           -  └─────────────────────┘
                         ↓
           - ┌────────────────────────────────────────┐
    Iterative│  PIE-family (ePIE,rPIE,mPIE/Iterative) │  →  非线性、迭代优化
           - └────────────────────────────────────────┘

```flow chart
                         ┌────────────────────────────┐
                         │   DPC-STEM (Differential   │
                         │   Phase Contrast)          │
                         │   线性、象限探测、提取相位梯度  │
                         └────────────┬───────────────┘
                                      │
                                      ▼
                         ┌────────────────────────────┐
                         │   CoM-STEM (Center of Mass)│
                         │   连续DPC，计算衍射质心 → ∇φ  │
                         │   线性近似、快速测梯度        │
                         └────────────┬───────────────┘
                                      │
           ┌──────────────────────────┴──────────────────────────┐
           │                                                     │
           ▼                                                     ▼
 ┌────────────────────────────┐                    ┌───────────────────────────┐ 
 │   SSB Ptychography         │                   │  WDD Ptychography         │
 │   (Single-Sideband)        │                   │  (Wigner Distribution     │
 │   弱相位、半线性；                                   Deconvolution)           
 │   扫描频率域分离干涉项；                              用Wigner分布做频域去卷积；  │
 │   得到复波函数(解析近似)                             仍为非迭代解析方法          
 └────────────┬───────────────┘                   └────────────┬──────────────┘
              │                                                │
         SSB/WDD 分支：频域解析方法（non-iterative），为ePIE 提供了理论基础
              └────────────────────────────┬───────────────────┘
              														 |
                                           ▼
                                      CDI-era
                                      Difference Map 通用相位恢复迭代框架
                                           │
                                           ▼
                      ┌─────────────────────────────────────────────┐
                      │PIE-family Algorithms实际上是DM的稳定化与特化形式
                      │                                             |
                      │  (PIE → ePIE → rPIE → mPIE → pcPIE → zPIE…) │
                      │  非线性、全波迭代；                            │
                      │  同时更新probe/object；                       
                      │  可处理厚样品、噪声、部分相干性                  │
                      └───────────────┬─────────────────────────────┘
                                      │
                                      ▼
             ┌────────────────────────────────────────────────────────┐
             │  Modern Extensions (2018–Now)                          │
             │  •  SR-PIE / Sub-pixel PIE (超分辨位移修正)              
             │  •  Momentum-PIE / 动量加速型                           
             │  •  ML / Deep Learning Ptycho (PtychoNN, PoCA,         │
             │     Ptychoformer)                                      │
             │  •  Differentiable / Hybrid Physics-DL Frameworks      │
             │  融合物理建模与可微优化，实现端到端重建                      
             └────────────────────────────────────────────────────────┘

```

//**事实上这些算法在数学上是有清晰逻辑的发展路线的，下个周我的工作重心是把这些理论推一遍。**

电子 ptychography 可以从两个角度划分：
 **(1)** 根据算法原理（频域 vs 实空间迭代）
 **(2)** 根据实验几何（Fresnel vs Fraunhofer, 4D-STEM 等）

------

###  按算法思想划分

| 类型                                                         | 代表算法                                                     | 特点                                                    | 是否需要迭代 | 是否是 SSB 类    |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- | ------------ | ---------------- |
| **A. 频域解析型 (Fourier-space analytical)**                 | **Single Side Band (SSB)**、Wigner-distribution deconvolution (WDD) | 在动量空间直接从干涉边带提取相位                        | ❌ 非迭代     | ✅ SSB 属于这一类 |
| **B. 实空间迭代型 (Real-space iterative)**                   | PIE / ePIE / rPIE / mPIE / pPIE                              | 在实空间和傅里叶空间来回约束求解（Gerchberg–Saxton 类） | ✅ 迭代       | ❌ 不属于 SSB     |
| **C. 混合模型 / 混合状态 (Mixed-state / partially coherent)** | Mixed-state PIE, WDD-ML, PtychoNN                            | 结合部分相干性建模、机器学习                            | ✅ 迭代       | ❌                |
| **D. 多层传播 (Multislice ptychography)**                    | Multislice-PIE, 3D STEM-ptycho                               | 适用于厚样品 / 多重散射                                 | ✅ 迭代       | ❌                |

### 按实验几何划分

| 几何类型                         | 描述                                 | 对应算法                              |
| -------------------------------- | ------------------------------------ | ------------------------------------- |
| **Far-field (Fraunhofer)**       | 探测器在样品傅里叶平面，收集衍射图样 | ePIE, rPIE, mPIE                      |
| **Near-field (Fresnel)**         | 探测器离样品有限距离                 | zPIE, Fresnel-PIE                     |
| **STEM / 4D-STEM 模式**          | 探针扫描，记录衍射盘重叠数据         | SSB, WDD, iterative STEM-ptycho       |
| **Bragg STEM / Nanodiffraction** | 高角度散射，晶体衍射条件             | Bragg-ptychography, multislice ptycho |

## The projection slice theorem

The 2D Fourier transform of a projection of a 3D object is equal to a *slice* through the 3D Fourier transform of that object.

It is also one of the principles of CT.

## 3 main stream transmission models

1. Projection approximation

2. Multislice propagation

3. Full Bragg/ dynamical diffraction

   ———————————–Other models—————————————

4. Multislice + partial coherence

5. WPOA

6. Dynamical diffraction/Bloch wave model

7. Multislice + inelastic scattering (absorptive model)

8. Non-paraxial/ beyond small-angle propagation

```flow chart
Propagation models in electron ptychography
│
├── (A) Linearized models
│     ├─ Weak Phase Object Approximation (WPOA)
│     └─ Projection approximation (2D phase object)
│
├── (B) Paraxial nonlinear models
│     ├─ Multislice propagation
│     └─ Multislice + partial coherence / absorption
│
├── (C) Nonparaxial or full 3D models
│     ├─ Full Ewald curvature (no expansion)
│     ├─ Dynamical diffraction / Bloch wave
│     └─ Bragg ptychography (selected reflections)

```



## A simple idea about the coded probe



## Single sideband ptychography

这是一个很细分化的领域。

是目前electon ptycho尤其是stem模式中最重要的相位重建与空间分辨率提升方法之一。

用stem-ptychography 时，我们在每个扫描点手机的是一个二维衍射图样，即4D-stem数据立方体。(4D:实空间扫描坐标和衍射图样的动量空间）

由于探测器的升级，所以相较于普通ptycho可以记录完整的衍射分布。

### 为什么要用4D-stem信息？

这是由于电子波的特性：高角度、多重散射、强相互作用，所以必须获得完整的角度分布（动量空间信息）

这些在可见光体系下通常不需要。

电子的波长极短，散射角可以达到数十毫弧度甚至上百毫弧度，而且电子与样品有强相互作用，导致多种散射模式同时出现。因此想要获得完整的波场信息，必须记录整个散射角空间的强度分布。

### 为什么电子体系需要这些信息？

#### 电子是强相互作用粒子（不是弱散射）

光学体系通常满足born近似，一次散射即可描述。

电子穿过样品会多次散射，多次传播。这使得不同角度的散射分量之间存在复杂干涉。

如果不完整记录角度分布，就无法做4D-STEM， 就会丢失多重散射的相干结构，无法正确反演样品电势。



#### 电子散射包含更丰富的动量转移信息

每个散射角对应一个动量转移q=kf-ki。

不同的q对应不同晶面或原子电势波矢。

光学体系的q范围很小（因为$\lambda$长），电子体系的q范围巨大。

所以电子探测器要能够覆盖很大的动量空间（高NA）才能提取出晶体结构、原子电势、应变、磁场等。



#### 电子波相干性结构复杂

电镜探针通常是带像差的汇聚波

不同入射角（孔径内不同点）对应不同平面波分量

他们在探测器上形成CBEDdisks（收敛束电子衍射盘）

当这些盘相互重叠的时候，会出现**相干干涉边带（side band）**

这些边带正是ptychography所利用的角度重叠信息。

## 4D-detector

### always fresnel

在衍射理论里，

- **Fraunhofer (远场)**：波前到探测面传播足够远，可视作**傅里叶变换**。
   条件是 Fresnel 数 $N_F = \frac{a^2}{\lambda z} \ll 1$。
- **Fresnel (近场)**：传播距离 z 不够长，仍要保留二次相位项。
   $N_F \gtrsim 1$。

#### 为什么电子 ptychography 几乎总在近场

波长极短 → “远场距离” 天文级长

- 300 keV 电子的波长 λ ≈ 2 pm；
- 若样品半径 a ≈ 1 µm，要满足 $N_F \ll 1$，
   需要 $z \gg a^2 / \lambda ≈ 0.5 m$。
   👉 电子显微镜里样品到探测器的距离一般只有 10 cm 以下，
   远达不到这个量级。
   所以电子衍射几乎永远处于 Fresnel 区（近场衍射）。

#### 不能像光学那样随意加大 z

光学系统里想进入远场，只要拉远屏幕就行；
 而电子显微镜的几何是固定的：样品、物镜、记录平面都在真空柱里几厘米范围内。
 再往远处传播，束流就散开、强度衰减、信号淹没。
 因此**几何限制 + 真空柱结构**决定了无法实现真正远场。

不过上述的一般适用于multislice.对于普通的stem-ptychography，还是会用lens变成远场传播。

对薄样品用 **单层远场模型**；

对厚样品用 **较少层的 multislice 模拟**；

很少去模拟整个样品外的近场传播。

##### 算法层面的思考：

| 模型类型                   | 方程形式           | 复杂度           | 是否常用             |
| -------------------------- | ------------------ | ---------------- | -------------------- |
| **Far-field (Fraunhofer)** | FFT                | O(N log N)       | 主流                 |
| **Fresnel single-step**    | FFT + phase + IFFT | O(3 N log N)     | 偶尔                 |
| **Multislice Fresnel**     | 多次传播           | O(L × 3 N log N) | 仅在强散射研究中使用 |

------

####  透镜“起反傅里叶变换”作用，但像差限制巨大

- 光学显微镜可用理想透镜实现物面 → 傅里叶面 → 像面 的变换，
   完全控制传播距离。
- 电子透镜是磁透镜：有强球差、像散、场畸变。
   它确实也在一定程度上完成傅里叶映射（物镜后焦平面≈倒易空间），
   但像差和散射使得这个“傅里叶变换”远非理想。
   电子 ptychography 正是要**在含像差的近场条件下反演波前**。

因此我们说“电子 ptychography 工作在近场”不是概念偷换，而是：

> 虽然透镜在做部分傅里叶变换，但样品-探测平面间始终处于 Fresnel 传播区。

#### Ptychography重要性

如果我们真的让电子波自由传播到 0.5 m（几乎理想远场），相位自然展开；

但我们用透镜把这段传播**光学压缩**到几厘米；

透镜必须引入一个强烈的 **二次相位项** 来聚焦这波前。
 这正是 defocus / spherical aberration 的物理来源。在 **ptychography** 中，透镜引起的像差会体现在**探针函数 $P(r)$** 的复相位结构上：
$$
P(r) = \mathcal{F}^{-1}\left\{A(k)\, e^{i\chi(k)}\right\}
$$
其中 $\chi(k)$ 就包含了你上面提到的各种相差。

==算法（例如 ePIE / rPIE）在重建时会**同时恢复样品和探针的复振幅**，==
 从而可以**数值地反推出并校正相差**。

所以即使透镜带来了相差，
 在 ptychography 中也不需要物理校正——算法能自动消除。

#### Multislice的特殊之处

透镜的相位透射函数 $e^{i \chi(k)}$ 只作用于**经过透镜口径的波前**，
 而 multislice 模拟描述的是**波在样品内部逐层穿透与传播**的过程。
 样品内部是物质势场 $V(x,y,z)$，不是自由空间，也不满足透镜方程。

→ 透镜无法“介入”样品内部的电子波传播过程。

在 multislice 模型中，样品被离散为 N 层：
$$
\psi_{n+1}(x,y) = 
\mathcal{F}^{-1}\Big[
\mathcal{F}\{\psi_n(x,y)\}
\, e^{i\pi\lambda \Delta z (k_x^2+k_y^2)}
\Big]
e^{i\sigma V_n(x,y)}
$$
这里的 $\Delta z$ 就是**相邻原子层之间的真实距离（几埃到几纳米）**。
 这个传播项代表电子波在样品实质厚度中**逐层前进**的相移与干涉，
 它没有任何可以“光学压缩”的自由度。

换句话说：

> 这段传播是“物理穿透”，不是“傅里叶变换”，所以透镜不能帮你缩短。
>
> **//未来会研究一下公式层面上和普通传播方程的区别**

与zPIE区分：

> **zPIE 的 “z” 指的是“探针与样品之间的传播距离 (defocus distance)” 的修正量。**

它不是样品厚度那种“几何意义的 z-axis 长度”，
 而是**波传播方向上的焦距误差 (defocus error)**。



------

#### 样品散射强 → 投影近似不再成立

电子与物质相互作用强，多重散射显著，
 这使得单纯的“远场 = 傅里叶变换”近似失效。
 必须采用 multislice 传播：逐层 Fresnel 传播 + 相位叠加。
 这本质上就是“近场多层传播”模型。

### Two overlap

真实的 STEM-ptychography（也称 4D-STEM）实验中：

- 探针依然会在样品上扫描（实空间 overlap）；
- 同时，探针内部的角度成分在探测器上也会发生盘重叠（k-space overlap）。

这两种重叠叠加起来，使得算法（ePIE、rPIE 等）能更好地重建相位。

这个两种overlap在electron pie family 以及SSB ptychography中都有。但是在optical ptychography 中主要还是依赖实空间overlap,k-space overlap弱或者近隐含存在。

## 在不同模型下，“两种 overlap”的意义完全不同

| 场景                        | 实空间 overlap 的作用                      | k-space overlap 的作用                                       |
| --------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **光学 PIE/ePIE**           | 提供强度冗余，用于解相位；没有它无法收敛。 | 几乎没有显著作用。                                           |
| **Electron SSB (4D-STEM)**  | 提供位置冗余                               | **是信息来源**：sideband 干涉项直接编码相位梯度。            |
| **Electron ePIE/rPIE/mPIE** | 提供扫描约束                               | k-space overlap 提供隐式约束（exit wave 相关性），但过强会导致反演不稳定。 |

所以要小心区分两种重叠的**角色差异**：

- 在 **SSB** 中，k-space overlap = 相位信息的来源。
- 在 **PIE 家族** 中，k-space overlap = 算法约束的隐式副产物。



### “那是不是因为不能用透镜减少传播距离？”

部分正确 ✅
电子透镜确实能做傅里叶映射，但受像差限制；
而且 STEM-ptycho 的探针本身就是透镜聚焦的结果，
再放一个透镜到样品后会干扰探针控制。
因此我们通常让样品后的波自由传播到探测器，
在算法中数值地“补偿”传播（反 Fresnel ）。

###  “如果缩短光学传播路径，是不是也能人为制造 k-space overlap？”

理论上 ✅ 可以部分实现。
如果你把光学系统调成 Fresnel 区（即近场），
探测器上会出现不同角谱分量的干涉条纹，
这确实会让“k-space overlap”显著增强。
这类实验就叫 near-field ptychography 或 Fresnel ptychography，
在 X-ray 和 visible 域都有研究。

不过代价是：

近场传播破坏了简单的傅里叶关系；

需要在算法中数值传播每一步；

对相干性和几何精度要求更高。

所以光学界一般只在特殊情况下（例如低相干或定量相位成像）才用 near-field 方案。

### 好的方面（信息冗余、有助重建）

1. **增强信息耦合性**
    不同扫描位置的衍射图样在 k-space 上共享部分频率成分 →
    算法更容易建立相位一致性。
2. **提升相位灵敏度**
    在 SSB 模型下，盘重叠越明显，干涉条纹越强，相位梯度信噪比更高。
3. **允许更小扫描步长**
    适度 k-overlap 对应更大的探针收敛角 α，可提升分辨率。

------

### 坏的方面（数值不稳定、混叠、解耦困难）

1. **强散射 & 大角度 → 盘重叠过度**
   - 当电子散射角大、像差存在时，不同角谱的干涉项相互混叠，
      导致“多重散射 + 角度耦合”问题。
   - PIE 的误差函数中，这会表现为非唯一解和伪收敛。
2. **算法过度约束**
   - 在 ePIE/rPIE 中，每个测量已经通过实空间 overlap 建立强约束。
      若再加上强 k-overlap，相位冗余会太多，反而使得梯度更新震荡。
   - 特别是在多 slice 模型中，k-overlap 过大意味着相邻层的散射信号在 detector 上混合，
      增加反演的“non-separability”。
3. **实验角度的代价**
   - 要得到明显的 k-space overlap，需要较大的聚光角 α；
      这往往伴随像差增加、剂量集中、SNR 降低。

数学解释：为什么过度 k-overlap 会变“坏”

在 ePIE 框架中，你要最小化：

![Screenshot 2025-11-01 at 6.37.30 pm](./electron.assets/Screenshot%202025-11-01%20at%206.37.30%E2%80%AFpm.png)

实空间 overlap 控制了 O(x) 与 P(x) 的相位一致性。

k-space overlap 让不同 j 的 I_j(k) 共享部分频率支撑。

如果盘重叠过大（比如 α 太大），那么：

各帧强度几乎相同；

误差函数的梯度方向趋于平行 → 优化难以前进；

导致 loss landscape 平坦 / multiple minima；

你就会看到重建开始震荡或停滞。

换句话说：

适度的 k-overlap 让约束更丰富；
过强的 k-overlap 会让约束彼此矛盾，信息线性相关，算法难以解耦。

## Detector

EMPAD（Cornell）是目前 electron ptychography 的标杆探测器：

- **像素阵列：** 128×128
- **像素尺寸：** 150 µm × 150 µm
- **动态范围：** $10^7$ e⁻ per pixel
- **帧率：** ~1 kHz
- **读出噪声：** < 1 e⁻ RMS
- **采样角度范围：** ±25 mrad (typical)

其内部为 hybrid pixel 架构：
 硅感应层 + ASIC 读出芯片，具备 *charge integration + counting* 混合模式。

EMPAD 的开发团队（Cornell D. Muller 组）在 2016 年 *Ultramicroscopy* 发表论文展示了**亚埃分辨率（<0.4 Å）**的电子 ptychography 重建。

Maybe I can try to develop higher-dimensional ptychography.

## 不同探测器的取舍

| 探测器              | 优点                   | 缺点             | 典型用途                    |
| ------------------- | ---------------------- | ---------------- | --------------------------- |
| EMPAD               | 动态范围极大，线性度高 | 帧率较慢，体积大 | 高精度 STEM ptychography    |
| Timepix3            | 单电子计数，时间分辨   | 动态范围小       | 低剂量，fast timing imaging |
| Merlin              | 平衡型，高速           | 较贵，需冷却     | 实验室 4D-STEM              |
| K3                  | 大面积、直探测         | 非线性、高噪区   | Cryo 4D-STEM / low-dose     |
| Dectris(非Electron) | 超低噪声               | 适配性弱         | X-ray / EUV CDI             |

Electron Ptychography

| 探测器                                               | 类型                                | 特点                                                         | 常见应用组/实验室                             |
| ---------------------------------------------------- | ----------------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| **EMPAD (Electron Microscope Pixel Array Detector)** | Hybrid pixel array (Cornell design) | ✅ 超高动态范围 (~10⁷ e⁻/pixel) ✅ 无读出噪声 ✅ 量子效率 >95% ✅ 精确线性响应 | Cornell D. Muller group（经典 e-ptycho 实验） |
| **MERLIN (Quantum Detectors)**                       | Hybrid pixel array (Medipix family) | ✅ 高帧率 (~10 kHz) ✅ 适用于 60–300 keV ✅ 模块化设计          | Cambridge / Diamond ePSIC                     |
| **Timepix3 / Medipix3**                              | Single-electron counting detector   | ✅ 能量分辨（time-of-arrival & time-over-threshold） ✅ ps–ns 时间分辨 ❌ 动态范围有限 | Delft / CERN / Amsterdam group                |
| **Cheetah / ASI / Gatan K3-IS**                      | Direct electron (CMOS)              | ✅ 高灵敏度、视频级帧率 (1000 fps +)  ❌ 容易饱和、非线性      | Low-dose cryo 4D-STEM                         |
| **PNDetector pnCCD**                                 | Monolithic pixel CCD                | ✅ 高DQE、低噪声、可用于中能STEM ❌ 动态范围中等               | pnDetector GmbH (Munich)                      |

## A classic difference:



diffraction theory:

$θ_c≈ \frac{λ}{D_s}$

**Van Cittert–Zernike theorem**



Bargg Crystallography:

$2dsinθ=nλ$

**Bragg’s Law**

## datasets

*Ophus et al.*, 4D-STEM datasets（Berkeley LBNL 公开）

*Maiden & Rodenburg* 模拟数据

