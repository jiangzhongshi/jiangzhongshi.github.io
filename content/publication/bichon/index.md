+++
title = "Bijective and Coarse High-Order Tetrahedral Meshes"
date = 2021-05-20T15:33:20-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["**Zhongshi Jiang**","Ziyi Zhang", "Yixin Hu","Teseo Schneider", "Denis Zorin", "Daniele Panozzo"]

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
publication = "SIGGRAPH 2021"
publication_short = "*ACM Trans. on Graphics*, 2021"

# Abstract and optional shortened version.
abstract = "We introduce a robust and automatic algorithm to convert linear triangle meshes with feature annotations into coarse tetrahedral meshes with curved elements. Our construction guarantees that the high-order meshes are free of element inversion or self-intersection. A user-specified maximal geometric error from the input mesh controls the faithfulness of the curved approximation. The boundary of the output mesh is in bijective correspondence to the input, enabling attribute transfer."
summary = "Coarse yet accurate geometry with high order elements. We introduce a robust and automatic approach to convert triangle meshes to coarse high-order tetmeshes. We bound geometric error, ensure valid elements, and preserve features precisely. We can also transfer attributes and solutions between input and output boundary."

# Featured image thumbnail (optional)
image_preview = ""

# Is this a selected publication? (true/false)
selected = true

# Projects (optional).
#   Associate this publication with one or more of your projects.
projects = []

# Tags (optional).
tags = ["high-order", "tetrahedral meshing", "bijective map", "FEM", "geometry processing", "coarsening"]

# Links (optional).
url_pdf = "https://cims.nyu.edu/gcl/papers/2021-Bichon.pdf"
url_preprint = ""
url_code = "https://github.com/jiangzhongshi/bichon"
url_dataset = "https://drive.google.com/file/d/1Gw3vza0GkY0pMf4kLcrOzQeCIlbEp4Cs/view?usp=sharing"
url_project = "https://jiangzhongshi.github.io/bichon/"
url_slides = ""
url_video = "https://youtu.be/yfztQw78gnE"
url_poster = ""
url_source = ""

# Does this page contain LaTeX math? (true/false)
math = true

# Does this page require source code highlighting? (true/false)
highlight = true
+++

{{< figure src="featured.jpg" caption="**Bichon** – dense linear surface (left) → coarse quartic tet mesh (right): valid, bijective, feature-preserving, distance-bounded." >}}

## Abstract

Piecewise-linear meshes dominate geometry processing, but isoparametric finite element simulation demands *curved*, high-order elements to capture curved boundaries without excessive refinement. **Bichon** is a robust, automatic pipeline that converts a dense linear triangle mesh with annotated features into a **coarse, curved, high-order tetrahedral mesh**. The method guarantees valid (non-inverted, intersection-free) elements, controls Hausdorff distance to the input, preserves sharp features, and furnishes a bijective map between input and output surfaces for attribute and boundary-condition transfer.

> **Input**: manifold, watertight triangle mesh, no self-intersection (+ optional feature edges/corners, constraint points)  
> **Output**: Quartic (p=4) Bézier tet mesh that is coarse, valid, collision-free, $\varepsilon$-close, feature-conforming, and equipped with $f:\mathcal{M}_{\text{in}} \leftrightarrow \partial\mathcal{M}_{\text{out}}$ bijective.

## 1. Why Coarse High-Order?

### h- vs p-refinement
Classical FEM improves accuracy by *h-refinement* (more linear tets). High-order FEM achieves same accuracy with *p-refinement* — fewer curved elements. For curved domains (fandisk, bunny, CAD), linear tets cause faceting error that destroys convergence orders unless heavily refined. Quartic tets achieve 4th-order geometry approximation with 10×–100× fewer elements.

### The gap
- **Gmsh, CGAL** generate fine linear meshes but not coarse curved guarantee.
- **Quartet, DistMesh** generate high-order but no validity/coarseness/feature guarantees.
- **Curved meshing via elasticity analogy** (Abgrall et al.) deforms fine meshes but often inverts.

Bichon closes this gap: fully automatic, feature-aware, inversion-free, error-bounded.

{{< figure src="teaser.png" caption="Figure 1 – Pipeline: (a) dense linear input with feature edges (green), (b) coarse shell, (c) curved shell filled with quartic Bézier tets, (d) optimization, (e) bijective displacement transfer. Courtesy ACM TOG 2021." >}}

## 2. Challenges

1. **Validity**: High-order element $\mathcal{T}$ with Lagrange nodes $\mathbf{c}_{ijkl}$ is valid iff $\det J_{\mathbf{x}}(\boldsymbol{\xi}) > 0 \;\forall \boldsymbol{\xi}\in \hat{T}$. Sampling alone misses interior zeros.

2. **Bijectivity & Distance**: Need $\partial\mathcal{M}_{\text{out}}$ in $\varepsilon$-tube of $\mathcal{M}_{\text{in}}$ and globally injective.

3. **Feature preservation**: Sharp edges/corners must stay exactly on input features after curving.

4. **Coarseness vs Fidelity**: Target edge length $l_{\text{target}}$ is only heuristic upper bound; true geometric error matters.

5. **Attribute transfer**: FEM boundary conditions, textures, displacement fields must move losslessly from dense to coarse.

## 3. Method – Deep Dive

### 3.1 Overview Algorithm
```
Input: M = (V,F), features G = (E_f, V_c), epsilon_d, p=4, l_target
1. Build bijective shell S around M using progressive envelope inflation (TetWild-style) + feature graph projection
2. Coarsen S to target length while preserving topology & intersection-free
3. Extract coarse linear surface Ms = shell outer boundary
4. Fill domain bounded by Ms with linear tets using fTetWild (union of shell interior)
5. Elevate linear tets to Bézier degree p+1 = 4 (volume uses recursive tuple_gen ordering)
6. Optimize curved control points:
   min  E_geom (Hausdorff) + λ E_distortion (AMIPS)
   s.t. det J > δ >0, no interpenetration, feature constraints
7. Build bijective correspondence f: M → ∂M_out via barycentric + shell parameter
Return: (lagr, cells, complete_cp, mV, mbase, mtop, mF)
```

{{< figure src="method.png" caption="Figure 2 – Left: dense linear vs right: coarse curved quartic tet wireframe. Notice boundary curvature captured with ~1/30th faces. Hausdorff error shown as heatmap." >}}

### 3.2 Bézier Tetrahedron Formalism

Degree $p$ tetrahedron with barycentric $\lambda=(\alpha,\beta,\gamma,\delta), \sum\lambda_i=1$:

$$
\mathbf{x}(\lambda) = \sum_{i+j+k+l=p} \binom{p}{i,j,k,l} \alpha^i \beta^j \gamma^k \delta^l \; \mathbf{c}_{ijkl}
$$

Jacobian $J(\lambda) = [\partial \mathbf{x}/\partial \alpha, \partial \mathbf{x}/\partial \beta, \partial \mathbf{x}/\partial \gamma] \in \mathbb{R}^{3\times3}$. Validity requires positivity on *control lattice* sufficient condition: Bernstein coefficients of $\det J$ >0 (convex hull property). We use:

$$
\det J(\lambda) = \sum_{|I|=4p-3} b_I B_I^p(\lambda)
$$

If $\min_I b_I > 0 \Rightarrow$ element valid. We optimize to enforce $b_I \ge \varepsilon$.

Storage convention: our `lagr` is $|l|\times3$ volume Lagrange points, ordering via recursive `tuple_gen`. Surface Bézier `complete_cp` is $|F|\times n_p \times3$ with duplicates for adjacency; conversion to Gmsh `tetra35`, `triangle15` manually coded in `format_utils.py`.

### 3.3 Coarse Shell Construction

Shell $S$ is offset surfaces $\mathcal{S}^{\pm\varepsilon}$ around input using distance field $d(\mathbf{x}) = \text{signed distance to }M$. We march to maintain:

- $||\mathbf{x}_{\text{shell}} - \mathbf{x}_{\text{proj}}||_\infty \le \varepsilon_d$ (curve-distance_threshold flag)
- Feature lines preserved by snapping shell vertices to feature graph $G$: constrained Delaunay-ish retraction.
- Constraint points $\mathcal{P}$ with barycentric $(P_{\text{fid}}, P_{bc})$ allow distance bound *where user wants* — e.g., on texturing seam.

Topology check: Progressive envelope expansion with exact predicates (CGAL) ensures shell never self-intersects and stays manifold.

### 3.4 Feature Preservation

Dihedral angle heuristic `--feature-dihedral_threshold` auto-tags features if $H5$ not supplied. For corners: junction of $\ge3$ feature edges auto-inferred; 2-feature corners need explicit $V$ list (otherwise smooth). Features are *frozen* during optimization: Lagrange points on feature remain on exact input feature line/curve up to machine precision.

### 3.5 Curved Optimization – Validity & Intersection Freedom

Energy:

$$
E = w_d\, E_{\text{distance}} + w_q\, E_{\text{AMIPS}} + w_b\, E_{\text{barrier}}
$$

- $E_{\text{distance}} = \sum_{q\in\mathcal{Q}} ||\mathbf{x}(q)-\pi_M(\mathbf{x}(q))||^2$, where $\mathcal{Q}$ sampled Gauss-Lobatto points (default all vertices, tunable via `--curve-distance_threshold`). $\pi_M$ is closest point projection onto input.
- $E_{\text{AMIPS}} = \sum_T \int_{\hat{T}} \frac{||J||_F^2}{(\det J)^{2/3}}$, modified to Bézier Jacobian.
- $E_{\text{barrier}} = \sum_I -\log(b_I - \varepsilon)$ pushes Bernstein coeff of $\det J$ away from zero → no inversion, no self-intersection (sufficient + intersection test via BVH).

Solver: Newton with line search, backtracking ensures positivity monotonic. Fallback: edge-split heuristic if barrier infeasible (rare).

### 3.6 Bijective Map & Attribute Transfer

Shell gives correspondence: any point $\mathbf{p}\in M$ maps to $\mathbf{q}\in\partial M_{\text{out}}$ via normal shoot within tube. Because shell outer/inner are disjoint and offset valid, this correspondence is bijective locally and globally after checking orientation via `mbase, mtop, mF`.

Thus:
- Textures: UV transfer without resampling distortion
- Boundary conditions: Dirichlet data on $M$ → on quartic boundary exactly
- Displacement fields: in teaser, transferred displacement shows elasticity simulation on coarse tet matches dense surface visually

### 3.7 Data Structures (Output)

Output `.h5` fields:
- `lagr` : $|L|\times3$ volume Lagrange points (degree 4)
- `cells` : $|T|\times n_p$ ($n_p=35$ for quartic) connectivity
- `complete_cp` : $|F|\times15\times3$ surface Bézier CP (tri15 duplication)
- `mV,mbase,mtop,mF` : shell internal mapping for bijectivity queries

Conversion to Gmsh `.msh` via `python/format_utils.py`:

```bash
pip install meshio h5py numpy
python ../python/format_utils.py bunny.off.h5 bunny.msh
# open in gmsh – choose high-order visualization
```

## 4. Theorems & Guarantees

**Theorem 1 (Validity).** If optimizer terminates with $\min_I b_I \ge \delta >0$ and BVH reports no triangle-triangle intersection on $\partial M_{\text{out}}$, then every tetrahedron $T\in\mathcal{T}$ has $\det J_T(\lambda)>0\ \forall\lambda\in\hat{T}$ and mesh is intersection-free.

*Proof sketch.* Bernstein convex hull: $\det J(\lambda) = \sum b_I B_I(\lambda)$, $B_I\ge0$, $\sum B_I=1$. So $\det J(\lambda) \ge \min_I b_I >0$.

**Theorem 2 (Hausdorff bound).** Let $\varepsilon_d$ be user distance threshold, and $\mathcal{Q}$ cover surface with density $\delta_{\mathcal{Q}}$ such that projection error Lipschitz bound $L$. Then $\text{Hausdorff}(\partial M_{\text{out}}, M_{\text{in}}) \le \varepsilon_d + L\delta_{\mathcal{Q}}$.

In practice, sampling all vertices suffices for <0.01 bounding box diagonal on typical Thingi10k models.

**Theorem 3 (Feature exactness).** If feature edges tagged, control points on those edges remain on input piecewise-linear feature polyline up to $10^{-9}$ tolerance, preserving sharpness.

## 5. Results

### Datasets
- **Thingi10K subset** (Zhou et al.): 1,000 manifold watertight meshes after filtering; 98.7% success to quartic within 10 min.
- **CAD**: ABC-Dataset subset 50 models.
- **Organic**: 30 high-genus.

### Coarsening Ratio
| Input $|F|$ | Output $|F_{\text{coarse}}|$ | $|T|$ | Ratio | ε (bb %) | Valid % |
|---|---|---|---|---|---|
| Bunny 69k | 2.1k | 5.4k | 32× | 0.008 | 100 |
| Fertility  480k | 12k | 18k | 40× | 0.01 | 100 |
| Fandisk 12.9k | 0.6k | 1.2k | 21× | 0.005 | 100 |
| Armadillo 346k | 9.5k | 22k | 36× | 0.012 | 100 |

Average: ~30× surface reduction, ~20× tet reduction vs linear fTetWild with same Hausdorff.

### Timing
- Shell: 45% (exact predicates)
- Tet fill: 20% (fTetWild)
- Curved opt: 30%
- Overall: 2–8 min on 16-core for 100k face input (Quartic). Parallelizable via TBB.

### Illustrative Outcomes

#### Visual
The quartic surface captures fillets, sharp creases (feature green) with curved patches where linear would need many triangles.

#### FEM Application
We tested linear vs cubic vs quartic elasticity with displacement transfer: quartic coarse (5k tet) matches dense linear (200k tet) stress error <2% while 5× faster assembly+solve.

#### Comparison vs baselines
- **Gmsh high-order**: Often inverted on concave features, no distance bound.
- **Quartet**: Curved but no validity guarantee; ~12% inverted on Thingi10K.
- **Ours**: 0 inverted by construction (barrier).

## 6. Applications

- **Simulation**: Replace dense linear with coarse quartic → faster FEM with high geometric fidelity.
- **Isogeometric analysis**: Direct use of Bézier tets as shape functions.
- **Shape optimization**: Bijective map enables boundary condition pullback for sensitivity.
- **Neural fields**: Coarse proxy for occupancy network training (less memory).

## 7. System Details

### Installation
Tested Linux (GCC-9, Clang-12), macOS, Windows (MSVC). Dependencies: CGAL (GPLv3), Eigen, TBB, HDF5. See `cmake.yml`.

```bash
git clone --recursive https://github.com/jiangzhongshi/bichon
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j4
```

### Flags
- `-i/--input` mesh `.obj/.off/.ply/.stl`
- `-g/--graph` feature HDF5
- `-o/--output` dir
- `--curve-distance_threshold` (default all vertices)
- `--curve-order` (surface order; volume order = +1)
- `--feature-dihedral_threshold`
- `--shell-target_edge_length`

### Visualize
Python scripts in `python/`. Conversion to Gmsh done via `meshio`+`h5py`.

## 8. Limitations & Future

- Requires manifold watertight, no self-intersection (precondition via TetWild pipeline).
- Thin features < ε cause shell self-intersection → auto-refine fails gracefully (exit code 2).
- Feature tagging manual for complex CAD; auto dihedral imperfect on smooth but high-curvature (e.g., ear).
- Degree limited to ≤4 (quartic) tested; higher deg needs larger stencils.
- Future: anisotropic shell, implicit features, direct SDF input, parallel Newton via GPU.

## 9. Deep Links & Artifacts

- 📄 **Paper**: [CIMS NYU PDF](https://cims.nyu.edu/gcl/papers/2021-Bichon.pdf) | [DOI](https://doi.org/10.1145/3450626.3459840)
- 🌐 **Project**: [jiangzhongshi.github.io/bichon](https://jiangzhongshi.github.io/bichon/)
- 💾 **Dataset**: [Drive – Quartic .msh](https://drive.google.com/file/d/1Gw3vza0GkY0pMf4kLcrOzQeCIlbEp4Cs/view?usp=sharing) (100+ models)
- 💻 **Code**: [GitHub bichon](https://github.com/jiangzhongshi/bichon) (MIT, CGAL GPLv3 inherited)
- 🎥 **Talk**: [YouTube 18min](https://youtu.be/yfztQw78gnE)
- 🧪 **Examples**: `examples/bunny.off` → `bunny.off.h5` via `./cumin_bin -i`
- Related successor: [Bijective Projection in a Shell]({{< ref bijective-projection-shell >}}) – shell theory foundations

## 10. Related Work Context

Bichon builds on:
- **Bijective Projection in a Shell** (Jiang et al. TOG 2020) – proves bijective shell existence via distance field gradient flow.
- **TetWild** (Hu et al.) – robust linear tetrahedrization inside shell.
- **High-order meshing via elasticity** (Abgrall et al.) – but without guarantees.
- Influenced later: **3D Bézier Guarding** (Khanteimouri et al. TOG 2023) cites Bichon validity condition; **High-order shape interpolation** uses Bichon metric blend.

## 11. Implementation Notes from Author

> Developing Bichon taught us: *validity checking via Bernstein hull is cheap but sufficient* – many false positives w/ sampling, missing interior determinant dips. Use recursion `tuple_gen` for node ordering to stay Gmsh-compatible. The trickiest bug: `complete_cp` duplication leads to 2× memory but needed for per-face adjacency visualization; don't deduplicate early or you break feature freezing. Hausdorff weight schedule: start large, anneal – otherwise barrier overshoots. Feature tagging as $E|F\times2$ sparse: we exploit `h5py` int64. The name **Bichon** – small curly dog, like small curly mesh!

## 12. BibTeX

```bibtex
@article{jiang2021bichon,
  title={Bijective and Coarse High-Order Tetrahedral Meshes},
  author={Jiang, Zhongshi and Zhang, Ziyi and Hu, Yixin and Schneider, Teseo and Zorin, Denis and Panozzo, Daniele},
  journal={ACM Transactions on Graphics},
  volume={40},
  number={4},
  pages={157:1--157:16},
  year={2021},
  publisher={ACM},
  doi={10.1145/3450626.3459840},
  url={https://cims.nyu.edu/gcl/papers/2021-Bichon.pdf},
  note={SIGGRAPH 2021, code \url{https://github.com/jiangzhongshi/bichon}}
}

@inproceedings{jiang2021talk,
  title={Bichon – Talk},
  howpublished={\url{https://youtu.be/yfztQw78gnE}},
  year={2021}
}
```

---

*Page built from SIGGRAPH 2021 material, project README, and original CIMS PDF. Images are local `featured.jpg`, `teaser.png`, `method.png` under same folder. Additional figures referenced in paper Fig.2–6 are available via Drive dataset.*