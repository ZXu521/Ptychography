### Research Note — Adaptive-z PIE (Zihan Xu)

**Date:** 2025-11-02
 **Author:** Zihan Xu

------

#### 📘 Title

**Adaptive-z PIE: A self-adaptive propagation-distance framework for electron ptychography**

------

#### 💡 Core Idea

In conventional electron ptychography, the propagation distance $z$ between the specimen and detector (or effective imaging plane) is fixed and identical for all probe positions.
 However, in realistic samples, the local **thickness** $t(x,y)$ and **scattering strength** $S(x,y)$ vary, leading to spatially dependent optical path differences that a fixed-$z$ model cannot fully represent.

**Adaptive-z PIE** proposes to introduce a spatially varying effective propagation distance:
$$
z_j = z_0 + \Delta z_j = z_0 + f(t_j, S_j)
$$
and use it inside the forward propagation operator:
$$
\Psi_j = \mathcal{P}_{z_j}\{ P(\mathbf{r}-\mathbf{R}_j)\,O(\mathbf{r}) \}
$$
where each scan position $j$ has its own effective $z_j$ automatically optimized during reconstruction.

------

#### ⚙️ Algorithmic Framework

1. **Outer loop** — update local $z_j$ to minimize intensity mismatch
   $$
   E(z_j) = \|\,|\Psi_j(z_j)| - \sqrt{I_j^{\text{meas}}}\,\|^2
   $$
   via binary search or gradient refinement.

2. **Inner loop** — standard PIE / ePIE update for object and probe.

3. **Optional regularization** — enforce smoothness of $z_j$:
   $$
   E_\text{total} = E + \lambda \|\nabla z_j\|^2
   $$

------

#### 🔬 Physical Interpretation

- $z_j$ represents the **effective propagation depth** compensating for local thickness or multiple scattering.
- The model acts as a **continuous approximation to the multislice formalism**, reducing computational cost while maintaining physical consistency.
- It bridges the gap between **Fraunhofer-based PIE** and **multislice ptychography**.

------

#### 📈 Expected Advantages

| Aspect             | Benefit                                                      |
| ------------------ | ------------------------------------------------------------ |
| Computational cost | Far lower than multislice                                    |
| Robustness         | Better convergence on thick / inhomogeneous samples          |
| Physical validity  | Still Fresnel-based; no empirical fitting                    |
| Applicability      | Ideal for medium-thickness or weak-to-moderate scattering samples |

------

#### 🧩 Relation to Existing Work

- Extends the idea of **z-PIE (optical)** — which globally optimizes a single propagation distance — into a **local, adaptive electron framework**.
- Functionally serves as a “lightweight multislice” approximation.

------

#### 📚 Future Directions

1. Simulate different thickness and scattering distributions to validate accuracy vs. full multislice.
2. Apply to 4D-STEM datasets and evaluate phase accuracy improvements.
3. Explore physics-guided or ML-regularized $z(x,y)$ estimation for faster convergence.

------

#### 🧠 One-sentence Summary

> *Adaptive-z PIE* introduces a self-adaptive propagation distance within the PIE framework, enabling thickness-aware and scattering-adaptive electron ptychographic reconstruction that bridges efficiency and physical realism.



               ┌──────────────────────────────┐
               │     Fraunhofer PIE / ePIE     │
               │------------------------------│
               │  • Fixed global propagation z │
               │  • Thin, weak-phase samples   │
               │  • Fast but less physical     │
               └──────────────┬───────────────┘
                              │
                              ▼
               ┌──────────────────────────────┐
               │        Adaptive-z PIE         │
               │------------------------------│
               │  • Local z(x,y) variation     │
               │  • Adjusted by scattering /   │
               │    thickness feedback         │
               │  • Continuous multislice-like │
               │  • Moderate accuracy & cost   │
               └──────────────┬───────────────┘
                              │
                              ▼
               ┌──────────────────────────────┐
               │         Multislice PIE       │
               │------------------------------│
               │  • Explicit multi-layer model │
               │  • Accurate multiple-scattering│
               │  • High computational demand  │
               └──────────────────────────────┘

Interpretation:

The vertical axis represents physical realism (↑ more accurate).

The horizontal axis represents computational complexity (→ heavier).

Adaptive-z PIE sits squarely in the middle — a continuous, self-adaptive approximation that retains Fresnel physics without the heavy cost of full multislice propagation.

接下来我需要系统学习一下multislice。

研究的关键是建立$z_{propagation} 和 z_{sample}$的映射。



