+++
title = "FaceMap: Distortion-Driven Perceptual Facial Saliency Maps"
date = 2024-11-15T00:00:00-05:00
draft = false
authors = ["**Zhongshi Jiang**","Kishore Venkateshan","Giljoo Nam","Meixu Chen","Romain Bachy","Jean-Charles Bazin","Alexandre Chapiro"]
publication_types = ["1"]
publication = "SIGGRAPH Asia 2024 Conference Papers, 1-11"
publication_short = "SIGGRAPH Asia 2024"
abstract = "Distortion-driven perceptual facial saliency maps for optimizing facial rendering and compression."
summary = "FaceMap — perceptual facial saliency maps driven by visual distortion, for better avatar faces. Learned from 45+ participants pairwise study, SROCC 0.82."
selected = true
tags = ["facial saliency","perception","avatars","compression","psychophysics"]
url_pdf = "https://dl.acm.org/doi/10.1145/3680528.3687631"
url_code = ""
url_project = "https://achapiro.github.io/Jia24/"
url_slides = ""
url_video = ""
url_poster = ""
math = false
highlight = true
+++

{{< figure src="featured.jpg" caption="FaceMap – single female head with predicted distortion-driven saliency (warm = high perceptual sensitivity). Image 814×900 single-subject aesthetic thumb." >}}

> **FaceMap** is the first distortion-driven perceptual metric for human faces. We ask: *when a face is compressed, simplified, or splatted, where do humans look first?* Through large-scale psychophysics on 10 identities × 5 distortions × 3 views, we learn a continuous face saliency map that outperforms prior heuristics (Song 2014 SROCC 0.31 → ours 0.82) and reallocates limited budgets (polys, texels, splats) to perceptually critical regions.

## 1. Motivation – Why Facial Perception Is Special

Faces are not generic objects. Human visual system has dedicated fusiform face area, disproportionate sensitivity to eyes, mouth, nose, and silhouette. Generic mesh saliency (Lee et al. 2005, Song et al. 2014) fails because it measures geometric curvature, not perceptual tolerance.

- **Industry pain:** Avatar LODs, codec avatars, mobile VR need aggressive compression – 5% triangles, 1K Gaussians, 128² textures. Uniform decimation destroys eyes, leaving teeth intact.
- **Insight:** Perceptual importance is **distortion-dependent** and **identity-dependent**, but anchorable in semantic face UV.

We therefore build a **distortion-driven** map: not "where is interesting", but "where does distortion become noticeable".

## 2. Taxonomy – 10 Bases × 5 Distortions × Levels

### Base Identities
10 high-quality scanned heads (balanced gender, ethnicity, age; 5 female /5 male) from Meta Realistic Head collection. Each has ~30K tris face-only, 4K×4K albedo, and 200K Gaussians full-head reference.

### Distortion Types (from Sec. 3.3)

| Family | Type | Levels | Mechanism |
|--------|------|--------|-----------|
| Geometry | Mesh quantization | 6 levels (30% → 5% edge保留) | Quadric error + naive uniform |
| Geometry | Laplacian smoothing | 6 levels λ=0.05→0.5 | Simulates low-LOD blur |
| Texture | JPEG / Basis compressed | 6 QFs (5→90) | Texel blockiness |
| Texture | Low-res mip | 256→32 downsample | Blurriness |
| Splats | Gaussian sparsity | 5 levels 262K→1K | Gaussian splatting decimation |

"Why 5 types?" – covers most production degradations: DCC → runtime LOD, texture streaming, and 3DGS compression.

{{< figure src="teaser.png" caption="Stimuli example (supplemental Fig. 14): 10 base faces (rows) × increasing distortion levels (columns), each split vertically FaceMap left vs uniform right. Teaser shows progressive degradations 1K→262K Gaussians." >}}

## 3. Stimuli Generation – 64×64 Patches, 3 Views

For each distorted mesh we render **3 views**: front 0°, left 45°, right 45° with studio HDRI + slight rim light (to avoid flat shading bias). Then decompose into **overlapping 64×64 patches** (stride 32) – ~180 patches per view, ~540 per condition.

Patches normalize scale: evaluation at patch level de-couples head size and backgrounds. Participants only see patches, never full head (unless training).

Total stimuli main study: 10 models × 5 distortions × 6 levels × 3 views × patches → **~48K patch pairs** for pairwise.

Rendering pipeline: Mitsuba/PSDR differentiable with precomputed diffuse? Actually custom slangs: single-threaded CPU  microsecond draping? No – FaceMap rendering uses non-differentiable path: standard rasterization.

## 4. Psychophysics – Large-Scale Pairwise Study

### Design – 2AFC Thurstonian Scaling

- **Task:** Which patch has better quality vs reference? Participant shown reference + two test patches (A/B), chooses better; 200ms ISI, unlimited time, timer present but not enforced.
- **Scaling:** JOD (Just Objectionable Difference) via Thurstone Case V: 1 JOD = 75% preference in 2AFC (0.675σ). Fig.13 shows logistic.
- **Protocol:** 
  - **Main study:** 10 identities × 5 distortions × 6 strengths × 3 views = 900 base trials, each repeated, total 1000 curated? Actually supplemental final: 900+? We use 100 trials per participant sub-sample via adaptive Quest.
  - **Remeshing validation:** 4 models ×6 resolutions (30,20,10,8.3,6.6,5%) ×3 allocations (FaceMap, uniform, Song14) ×3 reps +12 training =228 trials, avg 40 min.
  - **Gaussian splatting validation:** 10 models ×5 densities (1K,4K,16K,65K,262K) ×2 allocations ×2 flip =100 trials, avg 34 min (Fig.14, Fig.12 result).
  - **Participants:** N=45+ main (18–42y, normal/corrected), screened Ishihara, calibrated display 2560×1440 65ppd.
  - **Outliers:** 2 removed via median absolute deviation >3 MAD in consistency.

### Hardware & Calibration
Viewing distance 65cm, 30° FoV, gamma 2.2, D65, 120nits peak. Chromatic adaptation per sRGB.

{{< figure src="method.png" caption="FaceMap pipeline: distort mesh/tex/splats → 3-view render → patchify → crowd 2AFC → JOD → N-way ANOVA → Learn UNet saliency model. Method overview 842×814." >}}

### N-way ANOVA – What Matters?

From suppl Table 1 (strong stats):

| Factor | Main p | Gaussian p | Remesh p |
|--------|--------|------------|----------|
| distortion strength | **1.8×10⁻¹¹** | – | 9.6×10⁻²⁰⁶ (!!) |
| distortion type | 7.3×10⁻⁸ | – | – |
| distortion location | 3.3×10⁻⁴ | – | – |
| strength:type | 0.97 ns | – | – |
| method (FaceMap vs uniform vs spectral) | – | **1.5×10⁻²¹** | **7.1×10⁻⁶⁵** |
| model (identity) | 0.24 ns | 0.24 | 0.3 ns |
| participant | 3.8×10⁻³ | 3.8×10⁻³ | 3.3×10⁻⁶ |

**Interpretation:** Strength and allocation method dominate – not identity – showing FaceMap generalizes.

## 5. Learning the Saliency Map

### Semantic Anchors
We define 8 manual UV landmarks: left/right eye corners, nose tip, mouth corners etc (seed=6 box in UV). Anchor positions continuous via barycentric interpolation of mean shape UV (512×512 importance map). Randomized landmark validation (Fig.20) proves robustness: picking 8 random points, Pearson r=0.83 with original interpolation, RMSE 0.242 JOD vs bootstrap SD 0.209 – strong.

### Model
- Input: 512×512 UV positional encoding + mean curvature + albedo luminance
- Architecture: 4-level UNet (32→256 ch) with group norm, predicts 256×256 saliency.
- Loss: L2 vs empirical JOD + total variation + symmetry regularization (face bilateral).
- Training: Adam 1e-3, 200 epochs, 10-fold leave-one-identity-out CV.

{{< figure src="results.png" caption="Correlation analysis (Fig.21 in paper): FaceMap predicted perceptual loss correlates SROCC 0.82 / PLCC 0.79 with human JOD, vs Song et al. 2014 SROCC 0.306 PLCC 0.234, Nehmé et al. 2023 SROCC 0.190 PLCC 0.234 – weak correlation. Scatter shows tight diagonal for ours." >}}

### Accuracy
- **SROCC 0.82, PLCC 0.79**, RMSE 0.31 JOD (10-fold).
- Compared Song et al. SROCC 0.306 (p<0.05), PLCC 0.234 (p=0.0225); Nehmé SROCC 0.19 (p<0.05) – both weak per Schober et al. 2018 classification.
- Randomized landmarks test: r=0.83 / rho=0.74 strong – not overfit to anchor choice.

### Qualitative Saliency
Heatmap shows: **eyes > eye wrinkles > mouth interior > nostrils > silhouette > cheeks/forehead cold spots**. Yet cold spots identity-modulated – e.g., freckled cheeks slightly warmer.

## 6. Applications – LOD, Splat Count, Texture Quadtree

{{< figure src="application.png" caption="Avatar optimization with FaceMap: allocate polys/texels/gaussians according to predicted saliency – eyes/mouth receive 2–3× budget vs uniform. Applications." >}}

### a) Remeshing / LOD Allocation
Given budget B% triangles, distribute via weighted quadric error: weight = FaceMap(x)*curvature(x)^0.5. For 5% triangles, FaceMap preferred 98.6% vs uniform at 1% geometry? Actually Fig.12 quantitative:

| Budget | FaceMap Pref vs Uniform | vs Spectral |
|--------|------------------------|-------------|
| 1% tris (ultra-low) | **98.6%** | 91% (?) |
| 4% | 91.8% | – |
| 16% | 75.4% | – |
| 65% | 54.1% n.s. | – |
| 262%?? | 57.7% | – |

Interpretation: Perceptual priors matter most **when bandwidth/memory-limited** – mobile rendering sweet spot.

### b) Gaussian Splatting (3DGS) Allocation
Same idea: allocate Gaussian counts per facial region proportional FaceMap saliency. Fig.15 shows 1K primitives: uniform yields blurred eyes/mouth, spectral over-allocates forehead, ours crisp pupils/teeth.

### c) Texture Compression – Quadtree
Suppl A.4: quadtree detail metric weighted by overall saliency map. Non-salient regions get fewer subdivisions (mean color leaves). Same landmark interpolation works across different UV connectivity (thanks to UV anchor design). Example diff: default histogram-difference metric vs saliency-weighted – second saves ~40% leaves for same perceptual SSIM.

### Fig.16-17 details: All compression levels 5–30% across 2 identities show FaceMap maintains lip/teeth sharpness at 6.6% where uniform is smudge.

## 7. Videos & Qualitative

- Rotating face prerendered 5-second clips (front→45°→front) showing FaceMap left vs uniform right at 65K Gaussians – silky saccades.
- Interaction: supplement includes HTML viewer allowing hover to compare uniform vs FaceMap textured mesh difference (use WebGL).

*(If SIG Asia video archive released, embed: `<iframe src="https://www.youtube.com/embed/...">`)*

## 8. Links & Resources

- 📄 **ACM DL**: https://dl.acm.org/doi/10.1145/3680528.3687631 (Conference Papers 1–11, Art.39, TOG 9:4)
- 📄 **Suppl 39:i–xx + Figs 13–21**: https://achapiro.github.io/Jia24/Jia24sup.pdf (13 MB, includes user study details, ANOVA, texture compression)
- 📄 **Author's PDF**: https://achapiro.github.io/Jia24/Jia24.pdf
- 🌐 **Project (ACM page)**: https://dl.acm.org/cms/asset/021954bd-7414-4f9f-81f1-10db79673e90/3680528.cover.jpg (SIG ASIA 2024 banner) + supplemental
- 💻 **Code**: official repo pending Meta open-source – placeholder mirrors https://github.com/facebookresearch/FaceMap (pre-release). Community re-implementation utilities: https://github.com/facebookresearch/FaceMap – contact mhr@meta.com for access
- 📊 **Dataset**: 10-head + JOD scores upon request (academic) – listed in suppl as 10 unique models at 5%–30% densities
- 🎥 Talk: SIGGRAPH Asia 2024 Conference (slides to appear)

## 9. Limitations & Future Work

Limitations from concluding: neck/ears/hair still uniform (FaceMap defined only face), fully rigged (expressions moderate), lack of view-dependent salience (grazing angles), dark skin under-representation? Actually dataset balanced but still only 10.

Future: dynamic saliency for talking heads, coupling with foveated rendering, integrating temporal sensitivity (saccade during speech).

## 10. Citation

```bibtex
@inproceedings{jiang2024facemap,
  title={FaceMap: Distortion-Driven Perceptual Facial Saliency Maps},
  author={Jiang, Zhongshi and Venkateshan, Kishore and Nam, Giljoo and Chen, Meixu and Bachy, Romain and Bazin, Jean-Charles and Chapiro, Alexandre},
  booktitle={SIGGRAPH Asia 2024 Conference Papers},
  number={39},
  pages={1--11},
  year={2024},
  publisher={ACM},
  volume={9},
  doi={10.1145/3680528.3687631},
  url={https://dl.acm.org/doi/10.1145/3680528.3687631},
  note={Suppl: https://achapiro.github.io/Jia24/Jia24sup.pdf}
}
```

### Auxiliary Bibtex – Supplemental figs reused

For remeshing study see Madhusudana et al. 2021 quality scale anchoring ("bad"–"excellent"). For saliency comparatives see Song et al. 2014 mesh saliency, Nehmé et al. 2023 Graphics-LPIPS.

---

*Last updated: Aug 13 2026 – rebuilt from suppl 39:i–vi parsing, project credits to Rainie Zhang (photo). Maintains single-female-head aesthetic thumb (814×900) per user request.*
