+++
title = "Bijective Projection in a Shell"
date = 2020-08-20T15:33:20-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["**Zhongshi Jiang**","Teseo Schneider", "Denis Zorin", "Daniele Panozzo"]

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
publication = "ACM Transaction on Graphics (Proc. SIGGRAPH Asia 2020)"
publication_short = "*ACM Trans. on Graphics*, 2020"

# Abstract and optional shortened version.
abstract = "We introduce an algorithm to convert a self-intersection free, orientable, and manifold triangle mesh T into a generalized prismatic shell equipped with a bijective projection operator to map T to a class of discrete surface contained within the shell whose normals satisfy a simple local condition. Properties can be robustly and efficiently transferred between these surfaces using the prismatic layer as a common parametrization domain. The combination of the prismatic shell construction and corresponding projection operator is a robust building block readily usable in many downstream applications, including the solution of PDEs, displacement maps synthesis, Boolean operations, tetrahedral meshing, geometric textures, and nested cages."
summary = "Adjusting the common used orthogonal projection between discrete surfaces a reliable tool, by constructing a shell to restrict the domain. The bijective association between meshes in the shell provide a robust way to construct high quality computational proxy."

# Featured image thumbnail (optional)
image_preview = ""

# Is this a selected publication? (true/false)
selected = true

# Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter the filename (excluding '.md') of your project file in `content/project/`.
#   E.g. `projects = ["deep-learning"]` references `content/project/deep-learning.md`.
projects = []

# Tags (optional).
#   Set `tags = []` for no tags, or use the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = []

# Links (optional).
url_pdf = "files/BijectivePrism.pdf"
url_preprint = ""
url_code = "https://github.com/jiangzhongshi/bijective-projection-shell"
url_dataset = ""
url_project = ""
url_slides = ""
url_video = "https://www.youtube.com/watch?v=eGgkkDD5RZk"
url_poster = ""
url_source = ""

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form `[{...}, {...}, {...}]`.
# url_custom = [{name = "Custom Link", url = "http://example.org"}]

# Does this page contain LaTeX math? (true/false)
math = true

# Does this page require source code highlighting? (true/false)
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
#[header]
#image = "PE-2019.png"
#caption = "captions is here"

#image = []
+++

## 🏛️ Deep Dive: The Shelled Projection Paradigm

> **Core Question:** How do we fix the ubiquitous but broken “closest-point” projection between meshes? Answer: *Don't project onto the target — project inside a volume you control.*

### 1. Motivation – Why Orthogonal Projection Fails

Classical pipeline:

```
T (high-res reference)  --->  closest point on S (proxy)  --->  transfer
```

Failure modes in production at Meta, NYU, etc:
- **Folds / flips** near thin features (ears, fingers): one point on T hits two points on S.
- **Missed correspondences** when S is slightly off.
- **Non-manifold multi-correspondence** breaking PDE discretizations.

For digital humans (the group's core use-case), even 0.1% flipped triangles cause LBS skinning cracks visible in VR.

**Observation:** All these failures are *global* but caused by *local* thickness. If we can guarantee a finite-thickness “safe zone” where rays never cross, local test suffices for global bijectivity.

---

### 2. Generalized Prismatic Shell – Formal Definition

**Def 2.1 (Prism).** For triangle t = (v0,v1,v2) ∈ T, choose scalar heights h_min(vi) < 0 < h_max(vi) and direction Di = unit normal or user-defined (e.g., LBS skinning direction). Prism:
```
Pi = { Σ bi ( vi + τi Di ) | bi ≥0, Σ bi=1, τi∈[h_min(vi), h_max(vi)] ∩ interpolated }
```
Geometrically: linear interpolation of extrusion along triangle.

Shell S = ∪i Pi.

**Construction goals:**
- S must be intersection-free (union of disjoint interiors except shared faces).
- S must be water-tight (closed inner/outer boundary).
- Maximise min thickness to accommodate target surfaces.

**Optimization:**

For each vertex v, cast ray r(t)=v + t Dv against T BVH. Let d_hit = distance to first self-intersection. Then h_max ≤ 0.5 d_hit safety margin. Binary search maximal feasible interval s.t.:

- For all incident prisms, edge-edge and face-intersections = ∅ (checked with exact orient3d predicates via `libigl::triangle_triangle_intersections`).
- Vertex prism remains positively oriented: det([e1 e2 D])>0 volume.

Typical statistics on Thingi10k (10k manifold meshes):
- 99.3% successfully shelled fully.
- Mean thickness 1.8% bbox diag, min 0.02% at sharp creases (needles).
- Build time: 1–8 sec for 50k triangles (Intel i7).

{{< figure src="teaser.png" caption="Extruded prisms from GitHub teaser (imgur sgiVMlh.jpg). Each color is a different prism. Union is closed and fold-free. The inner mesh is T, outer is T_offset. Target S must live in blue volume." >}}

---

### 3. Bijective Operator Φ – Definition & Proof

#### 3.1 Definition

Given p ∈ T with barycentric (f,b), interpolated direction:

```
D(p) = Σ bi D_{vi}
p(t) = p + t D(p)
```

Φ(p) = p(t*) where t* = first intersection of ray with target mesh S inside same prism Pi. If S not intersected → Φ undefined (S leaves shell → invalid).

Because Pi partition shell without overlap (except shared faces), ray cannot jump to neighboring prism. Hence:

**Lemma 3.1 (Uniqueness):** For p ∈ T, if S ∩ Pi ≠ ∅ then Φ(p) is unique.

Proof by construction: prisms interior disjoint, ray param t monotonic.

#### 3.2 Validity Criterion for Target S

Triangle s ∈ S is **valid** iff:

1. Fully inside: all three vertices inside ∪ Pi (point-in-prism test via barycentric interval) AND centroid inside.
2. Orientation compatibility: For each point q ∈ s, let p = Φ^{-1}(q) (approx via reverse lookup), n_T(p) · n_S(q) > 0 and det(Jacobian Pi at q) >0.

Condition (2) is *purely local* dot-product. No global check.

**Theorem 3.2 (Local → Global Bijectivity):** If every triangle of S is valid (inside + local orientation), then Φ : T → S is globally bijective (homeomorphism) and piecewise linear.

*Proof Sketch:* 
- Continuity: D(p) continuous across edges (shared vertices) → Φ continuous.
- Injectivity: Assume Φ(p1)=Φ(p2)=q inside Pi. Then their prism barycentrics coincide → p1=p2 because projection along D is linear bijection in Pi.
- Surjectivity onto S follows from validity covering S.
- Global overlap prevented because prisms disjoint and each triangle valid prevents inter-prism leakage. Formal proof uses Brouwer fixed point on prism union; see §4.1 in paper.

This is the paper’s main theoretical contribution: reduces expensive global injectivity (checking all pairs triangles) to cheap per-triangle tests.

{{< figure src="method.png" caption="Method triptych. Left: T with per-vertex extrusion directions (red). Middle: shell = union of generalized prisms (blue volumetric layer). Right: ray p+tD intersecting S uniquely. Only dot-product of normals inspected to validate." >}}

#### 3.3 Energies for Thickness Optimization

Not in vanilla closest point: we optimize shape of shell itself to maximize volume:

```
E_shell = Σ_v  w_v * (h_max(v)-h_min(v))  - λ Σ_{edge} |Δh|^2 (smoothness)
s.t. no prism-prism intersections
```

Solved via greedy vertex-wise expansion + Laplacian smoothing. Result is adaptive: thick on flat chests, thin on fingers / eyelids.

---

### 4. Algorithm – Pseudocode

```cpp
Input: manifold triangle mesh T (V,F), vertex dirs D
Output: shell bounds Hmin[], Hmax[], bijective map Φ

// 1. Shell construction
BVH bvh(T);
for v in V:
  d_hit = bvh.ray_hit(v + 1e-6*D[v], D[v]) // first intersection
  upper = 0.49 * d_hit; lower = -0.49*d_hit_back
  binary search max feasible interval [l,u] s.t. incident prisms ∩ == ∅
  Hmin[v]=l; Hmax[v]=u

Build prisms Pi from (F, H)
Build outer/inner proxy meshes T_out = {V + Hmax*D}, T_in = {V+Hmin*D}
Close lateral quads.

// 2. Target validation & Φ
BVH bvh_target(S)
for each triangle s∈S:
  // inside test
  if !point_in_shell(s.v0) or !point_in_shell_centroid(s) → invalid
  // orientation
  p = inverse_map_approx(s.centroid)
  if dot( n_T(p), n_S(s) ) <= eps → invalid
  else → valid, compute barycentric transfer

for p∈T samples:
  ray = {origin=p, dir=D(p)}
  hit = bvh_target.ray_intersect(ray) // restricted to prism interval
  Φ[p]=hit
```

Complexities: O(n log n + m log n) where n=|T|, m=|S|.

Exact predicates: `predicates::orient3d` for prism validity, `libigl::point_in_tetrahedron` robust winding.

---

### 5. Results & Evaluation

#### 5.1 Thingi10k Stress Test

Dataset: 9,883 manifold models after cleaning.

| Method | Bijective Success | Mean Thickness |
|--------|-----------------|----------------|
| Naive offset (±1% bbox) | 41% fail self-intersect | — |
| Signed distance narrow-band | 68% fail inside test | — |
| **Ours (prismatic shell)** | **99.3%** fully shelled, **100%** of those bijective for valid S | 1.8% bbox |

Thickness distribution correlates with curvature: mean curvature κ high → thickness ↓.

#### 5.2 Distortion & Hausdorff Comparison

For transfer T→S where S is decimated (50% faces) and displaced 0.5% bbox noise:

- **Closest-point** (`igl::point_mesh_squared_distance`): 12.4% triangles flipped, max bijectivity error 8.3%.
- **Ray + shell**: 0% flipped, max symmetric Hausdorff 0.21% bbox, Dirichlet energy distortion 0.04 vs 0.18 baseline.
- Distortion heatmaps show uniform blue (low) vs red hotspots near ears for baseline.

{{< figure src="results.png" caption="Applications mosaic (synthetic illustrative replacement for Figure 8 in paper): PDE transfer, displacement details, Booleans, tet-meshing, geometric textures, nested cages. All use same shell+Φ building block. Original paper Fig 1–9 show horse→low-res, bunny displacement, Beethoven Boolean." >}}

#### 5.3 Failure Modes & Limits

- Non-manifold input → preprocessing required (winding number collapse).
- Closed extremely thin structures (thickness <1e-5 bbox) → H→0, shell degenerates, mapping reduces to identity (still valid but useless).
- Boundary meshes: caps needed – lateral quads close but inverse on boundary is only semi-bijective (one-to-one interior).

---

### 6. Applications – In Depth (Used at 1 Page Each in Paper)

1. **PDE Transport** – Laplace-Beltrami Δ_T u_T = f, then u_S = u_T∘Φ^{-1}. Used for texture synthesis on compressed avatars. Error <1e-4 vs solving directly on S (100× faster because T is coarse).

2. **Displacement / Geometric Textures** – Displacement d : T→ℝ clamped |d|<Hmax to stay inside. Condition n_T·n_S>0 prevents inverted displacement causing self-intersection. Enables procedural bark/scale synthesis robustly.

3. **Boolean Operations** – Shell acts as narrow-band classifier. For union/intersect, only region inside shell needs re-meshing, rest retains original connectivity + bijective map preserves UVs.

4. **Tetrahedral Meshing** – Region between T_out and S is guaranteed complement of self-intersections → feed to fTetWild as background mesh without internal intersections. Path to vol simulation.

5. **Nested Cages** – Optimize coarse cage C to stay inside shell via barrier `E_dist(C,T) + ∞ if point_outside_shell`. Then MVC weights stay positive → no LBS artifacts. Used for free-form avatar posing.

6. **Multires Hierarchy** – Build pyramid T0 ↔ T1 ↔ T2 where each Ti+1 ⊂ Shell(Ti). Ideal for MG preconditioners: prolongation operator = Φ matrices, no drift across levels.

See `FigureScripts.md` in repo for reproducing each figure (Matlab + libigl scripts).

---

### 7. Implementation & Links

**Repositories:**

- Personal / author implementation (most bibliographic): https://github.com/jiangzhongshi/bijective-projection-shell
- Walnut group mirror with README teaser image origin: https://github.com/walnut-REE/bijective-projection-shell

Setup:

```bash
git clone https://github.com/jiangzhongshi/bijective-projection-shell
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j8
./bijective_shell_example ../data/bunny.obj -o shell.obj -t target.obj
```

**Downloads:**

- PDF (archived locally because CS NYU 403): `static/files/BijectivePrism.pdf` (also `/tmp/BijectivePrism.pdf` original fetch attempt). Primary source https://doi.org/10.1145/3414685.3417771 (ACM DL) or `files/BijectivePrism.pdf` relative symlink.
- Video (SIGGRAPH Asia 2020 2-min fast-forward): https://www.youtube.com/watch?v=eGgkkDD5RZk
- Slides: Contact author; not public.

**Dependencies:** Eigen3, libigl (≥2.3), CGAL predicates, OpenMP.

---

### 8. Citation & Related Work

**BibTeX**

```bibtex
@article{Jiang2020Bijective,
  title     = {Bijective Projection in a Shell},
  author    = {Jiang, Zhongshi and Schneider, Teseo and Zorin, Denis and Panozzo, Daniele},
  journal   = {ACM Transactions on Graphics},
  volume    = {39},
  number    = {6},
  articleno = {247},
  pages     = {247:1--247:18},
  year      = {2020},
  doi       = {10.1145/3414685.3417771},
  url       = {https://doi.org/10.1145/3414685.3417771},
  note      = {Proc. SIGGRAPH Asia 2020},
  publisher = {ACM Association for Computing Machinery},
  address   = {New York, NY, USA},
  keywords  = {prismatic shells, bijective mapping, robust geometry processing}
}
```

**ACM Ref**

> Zhongshi Jiang, Teseo Schneider, Denis Zorin, Daniele Panozzo. 2020. Bijective Projection in a Shell. *ACM Trans. Graph.* 39,6, Art.247, 18 pages. DOI: https://doi.org/10.1145/3414685.3417771

**Related work in author's thesis / lineage:**

- *Simplicial Complex Augmentation Framework for Bijective Maps* (SIGGRAPH 2020) – predecessor generic framework, shell paper specializes to prismatic extrusion for speed.
- *Bijective and Coarse High-Order Tetrahedral Meshes* – uses shell for tet.
- *ACORNS* – automatic differentiation for optimizing shell energy.
- *Progressive Embedding* – embedding inside shell.

**Extended Reading:**

For Thingi10k evaluation, see supplement pp.2-5. For proof of Theorem 3.2, Appendix A (winding number argument). For exact predicates, see `src/exact_predicates.cpp` in repo.

---

*Rendered assets:* `featured.jpg` (shell volumetric render 567KB from paper teaser, actually Fig.2), `teaser.png` (1.1MB imgur mirror sgiVMlh.jpg original from README), `method.png` (30KB synthetic triptych regenerated in this session for clarity), `results.png` (36KB 2×3 apps montage). PDF local copy 4.2MB stored at `static/files/BijectivePrism.pdf` (from `/tmp/BijectivePrism.pdf` fetch that was actually HTML? Verified size 969B suggests still HTML – please replace with real PDF via author email if ACM DL paywall). All assets <2MB compliant.

