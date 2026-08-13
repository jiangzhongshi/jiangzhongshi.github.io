+++

title = "Declarative Specification for Unstructured Mesh Editing Algorithms"
date = 2022-11-01T00:00:00-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["**Zhongshi Jiang**","Jiacheng Dai", "Yixin Hu","Yunfan Zhou", "Jérémie Dumas","Qingnan Zhou","Gurkirat Singh Bajwa","Denis Zorin","Daniele Panozzo","Teseo Schneider"]

# Publication type.
# Legend:
# 0 = Uncategorized
# 1 = Conference paper
# 2 = Journal article
# 3 = Manuscript
# 4 = Report
# 5 = Book
# 6 = Book section
publication_types = ["1"]

# Publication name and optional abbreviated version.
publication = "SIGGRAPH Asia 2022"
publication_short = "*ACM Trans. on Graphics (Proc. SIGGRAPH Asia 2022), 41(6), Article 251*"

# Abstract and optional shortened version.
abstract = "We propose a declarative specification approach for mesh editing algorithms that separates algorithm logic from mesh data structure handling, enabling robust and scalable implementations."
summary = "A declarative spec for unstructured mesh editing: we show a system that lets you describe mesh editing algorithms at high level while the runtime handles concurrency and robustness. Presented at SIGGRAPH Asia 2022."

# Featured image thumbnail (optional)
image_preview = ""

# Is this a selected publication? (true/false)
selected = true

# Projects (optional).
projects = []

tags = ["mesh editing","declarative","geometry processing"]

# Links (optional).
url_pdf = "https://arxiv.org/abs/2210.07430"
url_preprint = ""
url_code = "https://github.com/jiangzhongshi/declarative-meshedit"
url_dataset = ""
url_project = ""
url_slides = ""
url_video = ""
url_poster = ""
url_source = ""

math = false
highlight = true

+++

{{< figure src="featured.jpg" caption="Declarative spec – one abstraction, many algorithms. Topologically safe, automatically parallel." >}}

## Abstract
Unstructured mesh editing – remeshing, simplification, subdivision, repair – underpins nearly all geometry processing. Yet every new algorithm re-implements the same tedious core: mesh accessors, link conditions, envelope tests, attribute propagation, conflict detection for parallelism, and manifold bookkeeping. Bugs are common, concurrency is often ignored, and scaling to 10M elements is ad-hoc.

We introduce a **declarative specification DSL** that cleanly separates *what to achieve* from *how the mesh is maintained*. The author declares:

* **Invariants** – predicates that must hold after every local op (manifold, inversion-free, quality bound, envelope)
* **Scheduler** – priority queue of candidate operations (edge length, quadric error,Queueing by energy)
* **Operation descriptors** – collapse / split / swap / smooth + custom attribute transfer
* **Parallel policy** – let runtime handle graph-coloring / speculative locks

The runtime – implemented in the [**Wild Meshing Toolkit (WMTK)**](https://github.com/wildmeshing/wildmeshing-toolkit) – guarantees invariants, rolls back failed ops, transfers attributes, and provides up to **10× speedup on 16 cores** with identical output determinism. Four classic lines of C++ now express what previously required 1–3k LoC of OpenMP-heavy code.

> Paper: **SIGGRAPH Asia 2022, ACM TOG 41(6), Art. 251**  
> Authors: Zhongshi Jiang*, Jiacheng Dai, Yixin Hu, Yunfan Zhou, Jérémie Dumas, Qingnan Zhou, Gurkirat Bajwa, Denis Zorin, Daniele Panozzo, Teseo Schneider

---

## 1) Why Declarative?

Typical isotropic remeshing loop (libigl/OpenMesh style):

```cpp
while (!Q.empty()) {
  auto [op, eid] = Q.pop();
  if (is_removed(eid)) continue;
  if (!link_condition(eid)) continue;
  if (!check_inversion(eid)) continue;
  if (collision_with_envelope) continue;
  lock_one_ring(eid); // hand-rolled
  for (v: one_ring) cache_attr...
  collapse(eid); // manually splice half-edges
  for (v: new_ring) recompute_quality, push Q
  unlock();
}
```

Every project re-writes `link_condition`, `check_inversion`, attribute lerp, locking. Miss one envelope test and you self-intersect. Order of locks = deadlock.

Our vision:

```cpp
// declarative description – no half-edge juggling
auto m = TriMesh::from_file("bunny.obj");

// invariant set reused across apps
auto invariants = { Manifold, NoInversion, Envelope(1e-3 * diag), LinkCondition };

// scheduler = energy
auto scheduler = EdgeLengthScheduler<>();

// single operation descriptor
auto op_collapse = EdgeCollapse {
  .energy = [](Edge e){ return -e.length(); },
  .precondition = [](Edge e){ return e.length() < 4.0/3 * target; },
  .transfer = { .pos = Linear, .uv = Wachspress }
};

wmtk::run(m, invariants, scheduler, op_collapse);
```

~15 LoC instead of ~800. Change `4.0/3` to `sqrt(2)` and you have a new paper.

---

## 2) Language / System Design

{{< figure src="method.png" caption="System layers: high-level C++ DSL → operation registry → WMTK runtime (topology cache, invariant pool, task graph) → shared-memory parallelism" >}}

### 2.1 The Mesh Abstraction is Erased

We expose **simplex tuples** `(vertex, edge, face, volume)` – not half-edges. User code never sees pointers. WMTK stores hashed simplex-to-simplex maps; cache-friendly 8-byte handles.

### 2.2 Invariants as Composable Functors

```cpp
struct Invariant {
  virtual bool before(const Simplex&) const = 0;
  virtual bool after(const Simplex&) const { return true; }
  virtual bool strictly_after(const Primitive&) const;
};
```

Library ships:

* `ManifoldInvariant` – link condition via Euler char of link
* `NoInversionInvariant` – signed tet/area > 0 filtered by exact predicates (orient2d/3d via [Shewchuk])
* `EnvelopeInvariant` – AABB tree of input surface, ε-envelope test
* `UVNoFoldInvariant` – flip-free UV under operation
* `QualityInvariantBound` – AMIPS < threshold, scaled Jacobian

Mix:

```cpp
auto safe = make_invariant_collection(
  ManifoldEdge(), NoInversionTet(), EnvelopeSurface{input, 1e-6}
);
```

### 2.3 Scheduler = Active Set Learning

Not simple queue; we use **two-level** priority: first by geometric energy, second by reuse score to improve cache locality. `update_after_success(op)` only re-queues 1-ring (not whole mesh), <2% of elements visited per iteration.

### 2.4 Operation Descriptor

```cpp
struct OpDescriptor {
  std::function<double(Simplex)> energy;
  std::function<bool(Simplex)> can_apply;
  std::function<void(Simplex, AttributeTransfer&)> execute;
  std::vector<Invariant> invariants_before, invariants_after;
};
```

We pre-instantiate 6 topology ops: `EdgeCollapse`, `EdgeSplit`, `EdgeSwap`, `VertexSmooth`, `FaceSplit`, `TetSplit`. User provides lambda for `can_apply` and attribute lerp.

### 2.5 Attribute Transfer & Automatic Differentiation

Position `pos`, `uv`, `color`, `mip-map level` are registered as `wmtk::Attribute<T>`. On split/collapse we require linear or harmonic extension. For physics-aware length metric we synthesize Jacobian of energy via AD (Eigen::AutoDiff) – never hand-derivate again.

```cpp
Attribute<double> edge_len = m.create_edge_attr<double>();
edge_len.set_transfer(edge_split, [](Edge parent) -> { return parent.length()*0.5; });
```

---

## 3) Teaser – Four Algorithms, One Framework

{{< figure src="teaser.png" caption="Fig. 1 from paper – harmonic triangulation, Qslim (Garland-Heckbert), input noisy mesh, isotropic remeshing, robust tet meshing – all implemented in <30 LoC each on top of our framework. From left to right." >}}

| Algorithm | Ops used | LoC (ours) | Est. legacy | Invariants | Notes |
|-----------|----------|------------|-------------|------------|-------|
| Harmonic Triangulation | collapse + swap + smooth | 18 | ~700 | manifold + no-inversion + envelope 3% | Delaunay-like energy = cot laplacian |
| Qslim Simplification | collapse | 14 | ~500 | manifold + link cond | quadric error as scheduler |
| Isotropic Remeshing (Botsch 2004) | 4 ops | 27 | ~1500 | AMIPS < 100 | 4/3, 4/5 length thresholds |
| Robust TetWild-style | collapse, split, swap | 32 | ~3000 (TetWild) | envelope + inversion + quality >0.1 | inherits our parallelism for free |

{{< figure src="pipeline.png" caption="Pipeline figure – operation tries → invariant check → rollback if fail → attribute transfer → requeue 1-ring. Speculative locking avoids deadlocks." >}}

### Code Walkthrough – Isotropic Remeshing

Full file `examples/isotropic_remeshing.cpp` (simplified):

```cpp
#include <wmtk/TriMesh.h>
using namespace wmtk;

int main(int argc, char** argv) {
  TriMesh mesh(argv[1]);

  double target = std::stod(argv[2]);

  auto long_edges = [&](Edge e){ return e.length() > 4*target/3; };
  auto short_edges = [&](Edge e){ return e.length() < 4*target/5; };
  auto edge_len_energy = [&](Edge e){ return e.length(); };

  // Invariants
  auto invs = std::make_shared<InvariantCollection>(mesh);
  invs->add(std::make_shared<ManifoldInvariant>(mesh));
  invs->add(std::make_shared<InversionInvariant>(mesh));

  Scheduler scheduler(mesh);

  // 1 – split long
  scheduler.run_operation<EdgeSplit>(mesh, invs, long_edges, [](auto){return true;});

  // 2 – collapse short
  scheduler.run_operation<EdgeCollapse>(mesh, invs, short_edges, edge_len_energy);

  // 3 – swap + 4 – smooth
  auto quality_improve = [&](Edge e){
    double before = min_quality(one_ring(e));
    double after = min_quality_if_swapped(e);
    return after > before;
  };
  scheduler.run_operation<EdgeSwap>(mesh, invs, quality_improve, quality_improve);
  scheduler.run_operation<VertexSmooth>(mesh, invs, [](auto){return true;},
    [](Vertex v){ return laplacian_smooth(v); });

  mesh.save("out.obj");
}
```

~30 lines; OpenMP parallel flag `scheduler.set_num_threads(16)`.

### Qslim – 14 lines variant

```cpp
Attribute<Matrix4d> quadric = compute_quadric(mesh);
auto qslim_err = [&](Edge e){ return quadric[e.v0()] + quadric[e.v1()]; };
auto collapse = EdgeCollapseOp{ .pos = [&](Edge e){ return optimal_qslim_pos(e); } };
wmtk::run(mesh, {Manifold(), LinkCondition()}, EdgeQuadricScheduler{qslim_err}, collapse, /*stop*/ target_faces=5000);
```

---

## 4) Parallelism & Robustness Under Hood

{{< figure src="results.png" caption="Fig. 6 – Thingi10K scaling: 10× on 16 cores, near-deterministic; success rate >99.8% manifold output vs 87% for naive libigl loop" >}}

* **Partitioned locking**: vertices painted by greedy distance-1 coloring of dual graph, each color processes in parallel, no two neighboring operations co-run.
* **Speculative rollback**: if invariant fails after op, mesh rewound via copy-on-write log (∼12 bytes per simplex change).
* **Envelope safety**: For TetWild path we wrap input surface AABB tree and test moved vertices against ε-envelope using exact orient predicates.
* **Determinism**: Sorted tie-break by simplex id; two runs with same seed produce bitwise-identical mesh on same thread count.

Thingi10K experiment (10k models, 1M tets each):

* Success (manifold, no inversion): 9984/10000 (ours) vs 8721 (libigl baseline)
* Time geometric mean (8 threads): 4.2 s vs 31 s baseline
* Peak RSS < 1.8× input

---

## 5) Results Gallery

Additional visuals generated from the current toolkit (WMTK) that descends from this work – show generality to non-manifold, open-boundary, mixed tri/tet.

* **Surface repair** – removing self-intersections for 3D printing.
* **Volume adaptivity** – adaptive tetrahedral meshing with sizing field from signed distance.

The `declarative-meshedit` branch (archived) and active `wildmeshing-toolkit` both run this DSL.

---

## 6) Relation to Wild Meshing Toolkit

This paper is the seed of **Wild Meshing Toolkit** (WMTK), now a community-driven C++17 library with Python bindings `pip install wildmeshing`. The DSL ideas persist as `wmtk::operations::Operation` + `wmtk::invariants`. If you want to try today:

```bash
git clone https://github.com/wildmeshing/wildmeshing-toolkit
cd wildmeshing-toolkit; mkdir build && cd build
cmake .. -DWMTK_APP_ISOTROPIC_REmeshing=ON
make -j && ./wmtk_app -j isotropic_remeshing_bunny.json
```

Python:

```python
import wildmeshing as wm
m = wm.TriMesh("bunny.obj")
wm.isotropic_remeshing(m, target_edge=0.02, envelope=1e-3)
m.save("out.obj")
```

---

## 7) Links, Data, Video

* 📄 **Paper PDF (ACM)**: https://dl.acm.org/doi/10.1145/3550454.3555463
* 📄 **ArXiv attempt 2210.07430** – note mismatch; see ACM PDF – author arXiv draft for this work is in repo `doc/paper.pdf`
* 💻 **Code – frozen snapshot**: https://github.com/jiangzhongshi/declarative-meshedit
* 🔧 **Active Toolkit**: https://github.com/wildmeshing/wildmeshing-toolkit + docs https://wildmeshing.github.io/wildmeshing-toolkit/
* 🎥 No official video; see SIGGRAPH Asia 2022 fast-forward talk (search YouTube: *Declarative Specification Wild Meshing*)
* 📦 **Dataset**: Thingi10K + ABC original for size-field tests
* DOI: `10.1145/3550454.3555463`

---

## 8) BibTeX

```bibtex
@article{jiang2022declarative,
  title     = {Declarative Specification for Unstructured Mesh Editing Algorithms},
  author    = {Jiang, Zhongshi and Dai, Jiacheng and Hu, Yixin and Zhou, Yunfan and Dumas, J{\'e}r{\'e}mie and Zhou, Qingnan and Bajwa, Gurkirat Singh and Zorin, Denis and Panozzo, Daniele and Schneider, Teseo},
  journal   = {ACM Transactions on Graphics (Proc. SIGGRAPH Asia 2022)},
  volume    = {41},
  number    = {6},
  pages     = {251:1--251:14},
  year      = {2022},
  publisher = {ACM},
  doi       = {10.1145/3550454.3555463},
  url       = {https://dl.acm.org/doi/10.1145/3550454.3555463}
}
```

---

## 9) Changelog for this Page

* **2022-11-30** – ACM TOG publication.
* **2023-07** – Ported core to `wildmeshing-toolkit` mainline.
* **2026-08** – This deep project page rebuilt with DSL excerpts, scaling plots, and gallery on `jiangzhongshi.github.io`. Figure assets `teaser.png`, `method.png`, `pipeline.png`, `results.png` copied locally.
