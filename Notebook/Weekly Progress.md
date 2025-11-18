# Weekly Progress Upgrade

1. Learned a new setup: put the aperture after the specimen.

   Investigated the principle of the setup, read the relevant chapter in fourier optics.

2. adaptive-z, Hessian-aware updates, scan-path optimization, mode automatic selection, uncertainty quantification

   These directions have seldom people work on it.

**自适传播距离 (adaptive-z)／局部厚度变化下的传播模型**
 虽然他们做了 multislice，但我没有看到“根据样品局部厚度／散射强度动态调整 z 距离”这一机制。
 → 你可以做：在多切片背景下，探讨“effective propagation distance”作为自适变量，并结合 ML 模型估计局部厚度。

**二阶优化／Hessian 信息驱动的更新策略**
 他们用了 LSQ-ML、多切片、贝叶斯调参，但“基于 Hessian 曲率信息来加速重建或调整步长”似乎还没成为主流。
 → 你可以提出：将 Hessian 或 LV (Laplacian-vector) 信息引入重建流程（例如 Newton-style 更新或者局部步长自适）

**扫描顺序／采集策略（active learning／主动采集）**
 自动参数选取是他们做的一部分，但主要关注重建参数。少见的是“根据当前重建质量／不确定性动态调整扫描顺序或探针位置”这种采集端优化。
 → 你可以在“采集 →重构”闭环中，设计一个“重构反馈 →下一扫描位置决策”模块。

**不确定性估计／重构置信区域**
 虽然他们强调高分辨率与低剂量，但我没看到很多像“每像素重构不确定度”、“模式系数的后验分布估计”这样的研究。
 → 你可以做：重建结果后估计不确定性、视觉化置信度 map，并探究其在实验流程优化中的作用。

**深度生成模型 + 物理驱动融合**
 他们虽有自动调参、LSQ solver，但“深度网络（如 VAE/扩散模型）＋物理正则化重建框架”这一方向目前公开资料中少见。
 → 你可以：提出一个 “Generator + Physics-Model (PLI + multislice) + LSQ-refine” 三段流程，专注于电子 ptychography。

Adam/lion