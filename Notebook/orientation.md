目前可以改进的方向：

1. 学习electron的噪声的类型。做低噪声优化或者滤波器。Stochastic minibatch PIE
2. Fast Partial Fourier Transforms warm-start PIE
3. Deep-Learning-Assisted Iterative PIE (Hybrid DNN + PIE)
   注意，必须不能够破坏物理一致性，仅仅在关键收敛上面做优化。此外，尽量physics-informed。



1. ### **Sub-gradient-projection based PIE variants**

   论文 Subgradient‑projection‑based stable phase‑retrieval algorithm for X‑ray ptychography（2024）提出 CRISP 算法，以 sub-gradient 投影方式提升 ePIE 稳定性。[journals.iucr.org](https://journals.iucr.org/j/issues/2024/04/00/nb5380/?utm_source=chatgpt.com)
    **对 EP 的启发**：

   - EP 的优化往往陷入 local minima / 模型偏差大，这种更强收敛稳定的方法是很有价值。
      **但改造挑战**：
   - EP 的 forward model 更复杂（多散射／厚样／mixed-state），sub-gradient 投影设计需重新推到电子模型层面。
   - 如果你已有强 ML 最优化框架，可能要衡量“新增投影机制”带来的实现负担 vs 收益。

2. ### **Live/real-time ptychography with projection-based methods**

   文章 Live Iterative Ptychography with projection‑based algorithms（2023）提出在扫描过程中实时重构、适应性扫描。[Emergent Mind](https://www.emergentmind.com/articles/2309.08639?utm_source=chatgpt.com)
    **启发**：

   - 对 EP 来说，如果你有 EMPAD 并行扫描／快速读出，也许可以考虑“在线重构＋反馈扫描”机制。
      **但实施难度高**：
   - 电子显微镜实验环境复杂（漂移、散射、非理想几何、厚样），实时控制和实时算法部署门槛高。
   - 除非你实验室可实现高速扫描 + 控制反馈，否则优先级可暂放。

3. 

