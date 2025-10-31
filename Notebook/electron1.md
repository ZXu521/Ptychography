1. The projection slice theorem

The 2D Fourier transform of a projection of a 3D object is equal to a *slice* through the 3D Fourier transform of that object.


It is also one of the principles of CT.

2. A simple idea about the coded probe
3. Single sideband ptychography

4. A classic difference:

   

   diffraction theory:

$θ_c≈ \frac{λ}{D_s}$

**Van Cittert–Zernike theorem**



​	Bargg Crystallography:

$2dsinθ=nλ$

**Bragg’s law**

5. datasets

*Ophus et al.*, 4D-STEM datasets（Berkeley LBNL 公开）

*Maiden & Rodenburg* 模拟数据

6. Two overlap

真实的 STEM-ptychography（也称 4D-STEM）实验中：

- 探针依然会在样品上扫描（实空间 overlap）；
- 同时，探针内部的角度成分在探测器上也会发生盘重叠（k-space overlap）。

这两种重叠叠加起来，使得算法（ePIE、rPIE 等）能更好地重建相位。