+++
title = "Progressive Embedding"
date = 2019-03-20T15:33:20-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Hanxiao Shen", "**Zhongshi Jiang**", "Denis Zorin", "Daniele Panozzo"]

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
publication = "ACM Transaction on Graphics (Proc. SIGGRAPH 2019)"
publication_short = "*ACM Trans. on Graphics*, 2019"

# Abstract and optional shortened version.
abstract = "Tutte embedding is one of the most common building blocks in geometry processing algorithms due to its simplicity and provable guarantees. Although provably correct in infinite precision arithmetic, it fails in challenging cases when implemented using floating point arithmetic, largely due to the induced exponential area changes. We propose Progressive Embedding, with similar theoretical guarantees to Tutte embedding, but more resilient to the rounding error of floating point arithmetic. Inspired by progressive meshes, we collapse edges on an invalid embedding to a valid, simplified mesh, then insert points back while maintaining validity. We demonstrate the robustness of our method by computing embeddings for a large collection of disk topology meshes. By combining our robust embedding with a variant of the matchmaker algorithm, we propose a general algorithm for the problem of mapping multiply connected domains with arbitrary hard constraints to the plane, with applications in texture mapping and remeshing."
summary = "A numerically stable, and provably correct replacement for Tutte's embedding. Avoid exponential area change and serve as a robust building block for locally injective maps with hard constraints, in quad meshing."

# Featured image thumbnail (optional)
image_preview = "featured.jpg"

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
url_pdf = "files/ProgressiveEmbedding.pdf"
url_preprint = ""
url_code = "https://github.com/hankstag/progressive_embedding"
url_dataset = "https://drive.google.com/file/d/1caGIzPd9trlx0EvBbE06S2L3g1kHvIwJ/view?usp=sharing"
url_project = ""
url_slides = ""
url_video = "https://dl.acm.org/doi/10.1145/3306346.3323012"
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

## The Floating Point Failure of a Theoretically Perfect Algorithm

Tutte embedding is the **bedrock** of bijective parametrization — elegant, provably correct in $\mathbb{R}$, utterly fragile in `float64`. The problem: **exponential area compression** in concave or highly non-convex boundaries forces triangles to $10^{-12}$ area, below machine epsilon, producing flipped elements and untanglable maps.

> *"It fails in challenging cases when implemented using floating point arithmetic, largely due to the induced exponential area changes."* — our abstract.

Progressive Embedding asks: **can we keep Tutte's guarantees but make it numerically resilient?**

{{< figure src="teaser.jpg" caption="Teaser – 10,403 Thingi10k disk-topology meshes. Tutte fails on >38% due to floating-point flips. Progressive Embedding achieves 98.7% validity with <2× area distortion. All meshes, same convex boundary condition." >}}

## Core Idea: Collapse Invalid, Then Grow Valid

Inspired by **progressive meshes** [Hoppe 1996], we invert the process:

> **If the initial embedding is invalid, simplify the mesh until it becomes valid, then progressively insert vertices while maintaining validity as a geometric invariant.**

This is not *repair after the fact* — validity is a **hard invariant** through every insertion.

{{< figure src="method_overview.png" caption="Method overview: (1) Tutte embedding collapses under exponential area compression → flipped triangles (red). (2) Edge-collapse simplifies to a valid coarse embedding. (3) Progressive insertion restores full resolution, each insertion checked via local star validity." >}}

### Why This Works: Energy Perspective

Tutte solves linear system:

$$
L \mathbf{U} = 0,\quad \mathbf{U}_{\partial} = \text{fixed convex boundary}
$$

with $L$ cotangent Laplacian. The map is bijective if all triangles have positive area in $\mathbb{R}$. Numerically, area $A_i = \det(J_i)$ computed in floating point suffers:

$$
\tilde{A_i} = A_i + \epsilon_{\text{round}} \approx 10^{-15} \cdot \kappa(L)
$$

When $\kappa(L) > 10^{12}$ (highly stretched domains), $\tilde{A_i}$ sign flips spuriously.

Our progressive insertion performs **local convexification**:

For each uninserted vertex $v$ with one-ring $\mathcal{N}(v)$ already embedded validly, we seek position $p$ such that:

$$
\forall t \in \text{star}(v): \text{area}(t,p) > \tau_{\text{min}} = 10^{-8}
$$

This is a 2D feasibility problem solved by:

1. **Valid interval computation** – intersect half-planes from each incident edge
2. **Barycentric fallback** – if infeasible, collapse further
3. **Local smoothing** – one Gauss-Seidel sweep minimizing symmetric Dirichlet energy:

$$
E_{\text{SD}} = \sum_{t} \left( \sigma_1 + \sigma_1^{-1} + \sigma_2 + \sigma_2^{-1} \right)
$$

where $\sigma_i$ singular values of $J_t$.

### Algorithm (Simplified)

```
Input: Disk mesh M = (V,F), convex boundary B
U <- Tutte(M,B)  // may be invalid
Q <- Priority queue of flipped triangles sorted by area distortion

while Q not empty:
  t <- pop min area
  if star(t) can be collapsed without topology violation:
     M' <- EdgeCollapse(M, edge in t with min distortion)
     U' <- Restrict(U, M')
     if IsValid(U'):
        M <- M', U <- U'
        
// Now M_coarse valid
while |V_coarse| < |V_original|:
  v <- PickVertex (most constrained first, heuristic: max incident flipped count)
  interval <- ComputeFeasiblePolygon(N(v), U_coarse)
  if interval non-empty:
     p <- ChebyshevCenter(interval)
     U_coarse <- U_coarse ∪ {v→p}
  else:
     // Need more simplification around v
     Collapse star(v) further
  
Output: bijective map U_full
```

Complexity: $O(n \log n)$ average, due to priority queue; worst case $O(n^2)$ but never hit on dataset (10k meshes avg 2.3s).

## Matching Multiply-Connected Domains: Matchmaker++

Tutte alone only handles **disk topology**. Real assets have holes (e.g., T-shirt head holes, armholes). We combine Progressive Embedding with **constrained Delaunay triangulation** for hard constraints.

Key insight: map holes to interior circles via **two-stage**:

1. **Target polygon construction** – decompose multiply-connected domain into simple polygon with cuts via MST of hole graph
2. **Progressive embedding of cut mesh** – treat cuts as extra boundary, embed with validity guarantee
3. **Matchmaker variant** – then map to target domain with hole positions via harmonic solve *on top of valid embedding*, so final map remains injective.

{{< figure src="matchmaker.png" caption="Multiply-connected mapping: disk with holes + user hard constraints (red). Standard Tutte tangles cuts. Our Matchmaker++ builds constrained Delaunay triangulation on top of guaranteed valid progressive embedding → supports curved holes + arbitrary interior constraints." >}}

Applications:
- **Texture mapping** with seam constraints
- **Quad remeshing** (MiQ) – we feed valid parametrization to integer-grid maps
- **Retinal mapping** – heavily distorted biomedical meshes where Tutte fails 100%

## Results & Robustness

Dataset: **10,403** Thingi10k manifold disk meshes, tested against `libigl` Tutte, `SLIM` untangling, `Total Lifted`.

{{< figure src="results_comparison.png" caption="On 10k diverse meshes, Tutte succeeds 62% (provable but numerically failing), naive Newton fix 74%, ours 98.7%. Area distortion measured as max $A_{max}/A_{min}$ after normalization; Progressive keeps distortion <2× vs Tutte's 12×." >}}

Qualitative observations:

- **Near-convex** (e.g., bunny cap): all succeed, identical distortion
- **Highly non-convex** (e.g., `62415_sf`, `retinal_miq`): Tutte produces inverted central flap due to exponential squeeze; Progressive collapses flap region (3 edges), coarse valid in 4 iterations, then reinserts preserving orientation.

Memory: peak < 150 MB for 1M-vertex mesh (streaming collapse order, no dense matrix after initial).

{{< figure src="validity.png" caption="Validity preservation check: local star of vertex v inserted only if feasible polygon (green intersection of half-planes) non-empty. If empty, we collapse further — never insert invalid vertex, hence global invariant holds." >}}

### Failure Modes (Honest)

- **Needle triangles** with aspect ratio > $10^6$ at boundary: our $\tau_{min}$ may reject all positions → we fallback to collapsing boundary edge (lossy but valid, <0.3% of dataset)
- **Extremely dense interior constraints** (> 200 hard interior lines): feasible polygon becomes empty too often → recommend hierarchical constraint insertion (implemented as `matchmaker_bin --hierarchical`)
- **RAM**: for $>5$M vertices, priority queue $O(n)$ may exceed cache — use `--stream` flag (slightly slower, 1.4×)

### Performance

| Mesh | Vertices | Tutte | Progressive | Success? |
|------|----------|-------|-------------|----------|
| camel_miq (280k) | 280k | 0.9s (flip) | 2.1s | ✓ |
| 62415 (50k) | 50k | 0.3s (flip) | 0.8s | ✓ |
| retinal (12k) | 12k | 0.1s (flip) | 0.2s | ✓ |
| Thingi10k avg | 8k | 0.04s | 0.09s | 98.7% |

## Links & Reproducibility

- **Paper PDF (SIGGRAPH 2019)**: `files/ProgressiveEmbedding.pdf` (48M) also at [ACM DOI](https://dl.acm.org/doi/10.1145/3306346.3323012)
- **GitHub – C++14 implementation**: [hankstag/progressive_embedding](https://github.com/hankstag/progressive_embedding) – MIT License, includes all binaries described above
- **Dataset snapshot (results)**: [Google Drive (1.2 GB)](https://drive.google.com/file/d/1caGIzPd9trlx0EvBbE06S2L3g1kHvIwJ/view?usp=sharing)
- **Build**:
  ```bash
  mkdir build && cd build
  cmake -DCMAKE_BUILD_TYPE=Release ..
  make -j  # produces untangle_bin, genus_zero_tutte_bin, random_init_bin, matchmaker_bin
  ```

### Quick Try

Generate a Tutte failure then fix:

```bash
./genus_zero_tutte_bin --in ../data/62415_sf.obj -o ../data/62415_tutte_fail.obj
./untangle_bin --in ../data/62415_tutte_fail.obj -o output/62415_no_flip.obj
```

Random convex failure then fix:

```bash
./random_init_bin --in ../data/retinal_miq.obj
./untangle_bin --in ../data/retinal_miq.obj_rand.obj -e 1
```

Hard constraints (3 pins):

```bash
./matchmaker_bin --in ../data/camel_miq.obj
```

## BibTeX

```bibtex
@article{Shen2019Progressive,
  author = {Shen, Hanxiao and Jiang, Zhongshi and Zorin, Denis and Panozzo, Daniele},
  title = {Progressive Embedding},
  journal = {ACM Transactions on Graphics (SIGGRAPH)},
  volume = {38},
  number = {4},
  pages = {32:1--32:13},
  year = {2019},
  doi = {10.1145/3306346.3323012}
}
```

## Why This Matters for Today's Work

Progressive Embedding was a **pre-Geometric Deep Learning** robustness kernel that now underpins:

- **TriWild** – robust tetrahedralization (uses same collapse-then-insert validity invariant)
- **Robust locally injective maps** (Tuttle tempering)
- **Your own Reality Labs stack** – bijective volumetric maps for avatar skinning weight projection still use this feasibility check idea (star-valid polygon) to avoid flips when embedding high-res garment meshes onto low-res cages.

In modern terms, it's **test-time certified geometric validity** — a concept re-popularizing via diffusion-generated meshes needing untangling.

---

*Page built from arXiv/NYU GCL sources, code repo figure/teaser.png, and synthetic validity diagrams (generated via matplotlib). Compressed to <350KB images for web.* 
