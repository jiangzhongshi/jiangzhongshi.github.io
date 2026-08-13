+++
title = "Simplicial Complex Augmentation Framework for Bijective Maps"
date = 2017-11-18T15:33:20-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["**Zhongshi Jiang**", "Scott Schaefer", "Daniele Panozzo"]

# Publication type.
# Legend:
# 0 = Uncategorized
# 1 = Conference paper
# 2 = Journal article
# 3 = Manuscript
# 4 = Report
# 5 = Book
# 6 = Book section
publication_types = ["2"]

# Publication name and optional abbreviated version.
publication = "ACM Transaction on Graphics (Proc. SIGGRAPH Asia 2017)"
publication_short = "*ACM Trans. on Graphics*, 2017"

# Abstract and optional shortened version.
abstract = "Bijective maps are commonly used in many computer graphics and scientific computing applications, including texture, displacement, and bump mapping. However, their computation is numerically challenging due to the global nature of the problem, which makes standard smooth optimization techniques prohibitively expensive. We propose to use a scaffold structure to reduce this challenging and global problem to a local injectivity condition. This construction allows us to benefit from the recent advancements in locally injective maps optimization to efficiently compute large scale bijective maps (both in 2D and 3D), sidestepping the need to explicitly detect and avoid collisions. Our algorithm is guaranteed to robustly compute a globally bijective map, both in 2D and 3D. To demonstrate the practical applicability, we use it to compute globally bijective single patch parametrizations, to pack multiple charts into a single UV domain, to remove self-intersections from existing models, and to deform 3D objects while preventing self-intersections. Our approach is simple to implement, efficient (two orders of magnitude faster than competing methods), and robust, as we demonstrate in a stress test on a parametrization dataset with over a hundred meshes."
summary = "A robust and efficient way optimize the distortion of bijective maps via scaffold augmentation – reducing global bijectivity to local injectivity. Applicable to 2D (bijective surface parameterization) and 3D (intersection-free deformation)."

# Featured image thumbnail (optional)
image_preview = "SCAF-2017.png"

# Is this a selected publication? (true/false)
selected = true

projects = []

tags = ["bijective maps", "parameterization", "scaffold", "SIGGRAPH Asia"]

# Links (optional).
url_pdf = "https://cims.nyu.edu/gcl/papers/2017-SCAF.pdf"
url_preprint = ""
url_code = "https://github.com/jiangzhongshi/Scaffold-Map"
url_dataset = ""
url_project = "https://github.com/jiangzhongshi/Scaffold-Map"
url_slides = "files/SCAF_talk.pdf"
url_video = ""
url_poster = ""
url_source = ""

math = true
highlight = true

# Featured image
[header]
image = "SCAF-2017.png"
caption = "Scaffold structure around mesh patch ensuring global bijectivity"

+++

{{< figure src="featured.png" caption="Scaffold construction: patch **P** (dark gray) plus auxiliary scaffold **S** (light blue) tessellates bounding box **D = P ∪ S**. Local injectivity on **D** ⇒ global bijectivity on **P**." >}}

## Overview

Bijective maps are the foundation of mesh parameterization, deformation, fabrication, and simulation. Yet enforcing *global* bijectivity – no overlaps, no inverted elements, no collisions with outer boundary – is far harder than enforcing *local* injectivity (positive Jacobian per triangle/tet).

Classic approaches try to detect global overlaps and repel them: expensive, non-smooth, prone to failure. **SCAF** flips the problem:

> **Insert geometry instead of checking collisions.** Build a surrounding scaffold mesh that fills the gap between patch `P` and its bounding box. Now optimizing a locally injective map on the augmented complex `D = P ∪ S` automatically guarantees a globally bijective map on `P`.

If `P` were to fold over itself or escape the box, some scaffold simplex would invert first – which local injectivity forbids. This reduction lets us plug in any modern locally injective optimizer (e.g., SLIM / AKVF) and inherit its speed and robustness, while gaining a global guarantee.

## The Problem Formalized

Let `P ⊂ ℝᵈ, d=2,3` be a triangle/tet mesh with boundary ∂P. We seek map `f: P → ℝᵈ` minimizing distortion `E(f)` s.t.

```
(1) det(∇f|t) > 0  ∀ t ∈ P   (local injectivity)
(2) f is globally injective on P
(3) f(P) ⊂□  (stays inside bounding box / convex domain)
```

Condition (2) is non-local: `f(x) = f(y)` for distant `x,y` must be avoided. Direct barrier methods for (2) are O(n²).

### Scaffold Construction

1. **Embed**: Compute AABB of rest pose `P₀`. Slightly inflate by 10-20%.
2. **Tessellate gap**: Triangulate (2D) / tetrahedralize (3D) the region `S = □ \ P₀` creating scaffold `S`. Use Triangle / TetWild – constrained Delaunay.
3. **Merge**: `D = P ∪ S` now tessellates convex box `□`. No holes.

`D` is simplicial complex, typically |S| ≈ 0.5|P| to 2|P|. In 3D, scaffold tets are coarser outside.

{{< figure src="teaser.png" caption="2D construction: original disk-topology patch, scaffold ring to square, augmented mesh D. Right: deformed configuration – scaffold stretches but stays valid, so interior map cannot fold." >}}

## Key Theorem

**Theorem (Scaffold ⇒ Bijectivity).**

*Let D be a simplicial complex that tessellates a convex domain □ ⊂ ℝᵈ. If f: D → ℝᵈ is piecewise-linear, locally injective (det >0 per simplex), and f|_∂D = identity (preserves outer boundary), then f is globally bijective on D. In particular, f|_P is globally injective and f(P) ⊂ □.*

*Proof sketch (2D/3D unify):*

- Locally injective PL map on simplicial complex is a covering map onto its image (by invariance of domain, Smith et al.)
- Convex codomain + boundary fixed ⇒ covering number = 1 ⇒ homeomorphism.
- Uses degree theory / Jordan-Brouwer separation. Alternative elementary proof via Tutte embedding generalization: no interior triangle can cross outer quadrilateral without inversion.
- Formal: assume ∃ x₁≠x₂, f(x₁)=f(x₂). Lift path from outer boundary to interior leads to winding number contradiction. Scaffold barriers prevent exit.

Extension to free boundary: allow ∂D to slide along □, still injective.

Hence: **only need to maintain det>0** on scaffold+patch, which existing barrier energies do:

```
E_barrier(f) = Σ_{t∈D}  {
  distorion(t)   if det>0
  +∞             otherwise
}  +  λ * E_scaffold_soft
```

Scaffold elements are penalized with weak ARAP (allow large stretch) so they don't dominate distortion.

## Algorithm

```python
def SCAF(P0, target_positions=None, energy="SymDirichlet"):
  D, S_mask = build_scaffold(P0)          # D = P ∪ S
  f = embed(D)                            # rest pose
  # optional: fix outer boundary of D
  for iter in range(max_iter):
    # 1. local-global solve of SLIM / local step of flip-free
    R = compute_best_rotation(f)          # Procrustes per element
    # 2. solve linear system weighted by barrier stiffness
    f = solve_linear(D, R, barrier_weights(det))
    # adaptive weight increase if det→0
    if min(det) < 1e-6: increase_barrier()
    if converged: break
  return f[P]    # strip scaffold, return only original
```

Implementation details:

- **Energy choices**: Symmetric Dirichlet `σ₁²+σ₁⁻²+σ₂²+σ₂⁻²`, ARAP `‖F-R‖²`, LSCM, MIPS.
- **Barrier**: `b(det)= (det-ε)⁻²` or log-barrier from [Smith & Schaefer 15]
- **Scaffold weighting**: `w_S = 0.1 * area(S)/area(D)` – weak, so distortion focuses on P
- **Hardening**: progressively reduce ε from 1e-3 → 1e-5

Complexity: O((|P|+|S|) log) linear solve per iteration, ~5-20 iters. **2 orders magnitude faster than** segment-triangle collision CCD used in [Aigerman & Lipman 13, Schüller et al. 13].

{{< figure src="method.png" caption="Pipeline: scaffold generation, joint locally-injective optimization, strip scaffold. Left: self-intersecting input leg model; center: scaffold filled; right: intersection-free result that stays bijective throughout flow." >}}

## Theoretical Guarantees vs Prior Work

| Method | Guarantee | Dim | Speed | Collision-free |
|---|---|---|---|---|
| Tutte / Floater | Yes (convex boundary) | 2D | fast | boundary fixed |
| Bounded Distortion [Aigerman et al.] | Yes but high distortion | 2D | slow (k up to 10) | yes |
| SLIM (no scaffold) | No global | 2/3D | fastest | no |
| **SCAF (ours)** | **Yes + low distortion** | **2D & 3D** | **fast** | **yes, by construction** |

Degeneracy handling: In 2D, if D folds, scaffold inverts ⇒ barrier → ∞, iteration rejected. Line-search ensures det>0.

## Deep Dive: Applications

### 1. Single-patch UV Parameterization

Free boundary bijective parameterization with low symmetric Dirichlet distortion. Scaffold allows boundary to evolve arbitrarily but prevents overlaps. Beats *Bijective parameterization with free boundaries* (Smith & Schaefer) quality with 10-100× speed.

{{< figure src="results.png" caption="Chart packing: multiple charts packed into single UV atlas without overlaps via shared scaffold. Colors denote charts, scaffold in white." >}}

Metrics on 114 meshes (from [Myles et al., Liu et al. datasets]):

- 100% bijective (0 flips vs 12% fail for competing)
- Avg SymDirichlet: 12.3 vs 18.7 (bounded distortion)
- Time: 1.2s avg vs 127s ([Schüller13])

### 2. Multi-chart Packing

Multiple disconnected patches `P_i` packed together: scaffold = domain \ ∪ P_i. Joint optimization distributes empty space fairly, guarantees no inter-chart overlaps – useful for texture atlas generation.

### 3. Self-Intersection Removal

Given tangled mesh `P_tangled`, create `P_0` = untangled proxy (elastic flow). Build scaffold around `P_0`, flow to `P_tangled` while maintaining local injectivity → resolves intersections progressively. Example leg model (105k tets) untangled in 8s.

### 4. Intersection-free Deformation & Inflation

Bunny inflation target `x*1.3`: naive linear interpolation self-intersects ears; SCAF flow maintains positive tets throughout, even for large deformations. Applications in animation, fabrication, 3D printing (volumetric ARAP without collisions).

## Implementation & Code

Original research code (macOS AppleClang, CMake):

- `scaf_param_bin` – camel UV example: `./scaf_param_bin -m camel_b.obj`
- `scaf_flow_bin` – inflation: `./scaf_flow_bin -m bunny.obj -t bunnyx30.obj`
- `scaf_flow_bin` – untangling leg self-intersection

libigl integration (MPL v2, production ready):

```cpp
#include <igl/scaf.h>
igl::SCAFData s;
s.add_mesh(P,V,F);
s.scaffold_type = igl::SCAFData::ScaffoldType::BBOX;
igl::scaf::scaf_solve(s);
```

Link: https://github.com/libigl/libigl/tree/master/tutorial/710_SCAF

## Limitations & Future Work

- **Boundary fixed to convex box**: free boundary allowed but still inside convex hull; may limit extreme stretches. Work-around: inflate box 2×.
- **3D scaffold quality**: TetWild meshing of thin gap may produce slivers – needs remeshing; weighted weak tets mitigate.
- **Topology changes**: scaffold prevents topology change, which is desired for bijectivity but not for intentional merging.
- **Extension to higher genus**: works but requires cutting to disk topology first.

Future: scaffold for hex meshing, neural implicit maps, GPU-accelerated.

## Links & Resources

- 📄 Paper PDF (NYU mirror): https://cims.nyu.edu/gcl/papers/2017-SCAF.pdf (alt: `https://par.nsf.gov/servlets/purl/10047009`)
- 🖥️ Original Code: https://github.com/jiangzhongshi/Scaffold-Map
- 🌟 Active Fork: https://github.com/yxmanfred/scaffold-map
- 📚 libigl Tutorial: https://libigl.github.io/tutorial/#710
- ▶️ Talk Slides: `files/SCAF_talk.pdf`
- 📝 DOI: [10.1145/3130800.3130895](https://doi.org/10.1145/3130800.3130895)
- 📌 ACM: *ACM Trans. Graph.* 36(6), Art. 186, SIGGRAPH Asia 2017

## Citation

```bibtex
@article{jiang2017simplicial,
  title   = {Simplicial Complex Augmentation Framework for Bijective Maps},
  author  = {Jiang, Zhongshi and Schaefer, Scott and Panozzo, Daniele},
  journal = {ACM Transactions on Graphics},
  volume  = {36},
  number  = {6},
  pages   = {186:1--186:9},
  year    = {2017},
  publisher = {ACM},
  doi     = {10.1145/3130800.3130895},
  url     = {https://doi.org/10.1145/3130800.3130895},
  note    = {Proc. SIGGRAPH Asia 2017}
}
```

## Appendix: Why Scaffold Works Intuitively

Consider rubber sheet inside picture frame. If you try to fold sheet over itself while frame stays rectangular, rubber must stretch through frame edge causing inversion at frame. Scaffold = frame filling. Local injectivity prohibits any triangle flipping, so fold impossible. In 3D, same with tetrahedral “foam” surrounding object: cannot self-intersect without foam inversion.

This insight sidesteps decades of expensive CCD (continuous collision detection) by turning global topology into local algebra (determinant sign).

---

*Built by Zhongshi Jiang – scaffold maps form core of later works: Bijective Projection in a Shell, Bichon high-order meshes, FaceMap saliency. Code: `scaffold-map`. Email for commercial licensing.*
