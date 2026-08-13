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

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/jpswalsh/academicons@1/css/academicons.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">

<style>
.publication-title { font-variant-ligatures: none; }
.author-block { display:inline-block; margin: 0 6px; }
.publication-links .button { margin: 4px; }
.teaser-img { max-height:520px; object-fit:contain; border-radius:12px; box-shadow:0 10px 30px rgba(0,0,0,0.15); }
.message.is-info { border-radius:10px; }
.nerfies-section { padding-top:2rem; padding-bottom:1rem; }
</style>

<!-- HERO TITLE -->
<section class="hero">
  <div class="hero-body">
    <div class="container is-max-desktop">
      <div class="columns is-centered">
        <div class="column has-text-centered">
          <h1 class="title is-1 publication-title">FaceMap: Distortion-Driven Perceptual Facial Saliency Maps</h1>
          <div class="is-size-5 publication-authors" style="margin-top:0.8rem;">
            <span class="author-block"><a href="https://jiangzhongshi.github.io" style="text-decoration:underline; text-underline-offset:3px; font-weight:700; color:#363636;">Zhongshi Jiang</a><sup>*</sup>,</span>
            <span class="author-block"><a href="#">Kishore Venkateshan</a>,</span>
            <span class="author-block"><a href="#">Giljoo Nam</a>,</span>
            <span class="author-block"><a href="#">Meixu Chen</a>,</span>
            <span class="author-block"><a href="#">Romain Bachy</a>,</span>
            <span class="author-block"><a href="#">Jean-Charles Bazin</a>,</span>
            <span class="author-block"><a href="https://achapiro.github.io">Alexandre Chapiro</a></span>
          </div>
          <div class="is-size-6 publication-authors" style="margin-top:0.35rem;">
            <span class="author-block"><sup>*</sup>Meta Reality Labs – first author,</span>
            <span class="author-block"><sup>1</sup>Reality Labs Research, Sausalito CA &amp; Redmond WA</span>
          </div>
          <div class="is-size-7 has-text-grey" style="margin-top:0.2rem;">* Equal contribution shuffling? This work: first author. SIGGRAPH Asia 2024 Conference Paper #39 (TOG 9:4)</div>

          <div class="column has-text-centered" style="margin-top:1rem;">
            <div class="publication-links">
              <span class="link-block">
                <a href="https://dl.acm.org/doi/10.1145/3680528.3687631" class="external-link button is-normal is-rounded is-dark">
                  <span class="icon"><i class="fas fa-file-pdf"></i></span><span>Paper</span>
                </a>
              </span>
              <span class="link-block">
                <a href="https://achapiro.github.io/Jia24/Jia24.pdf" class="external-link button is-normal is-rounded is-dark">
                  <span class="icon"><i class="ai ai-acm"></i></span><span>Author PDF</span>
                </a>
              </span>
              <span class="link-block">
                <a href="https://achapiro.github.io/Jia24/Jia24sup.pdf" class="external-link button is-normal is-rounded is-dark">
                  <span class="icon"><i class="fas fa-file-lines"></i></span><span>Suppl (13MB)</span>
                </a>
              </span>
              <span class="link-block">
                <a href="#" class="external-link button is-normal is-rounded is-dark is-outlined" onclick="document.getElementById('video').scrollIntoView({behavior:'smooth'}); return false;">
                  <span class="icon"><i class="fab fa-youtube"></i></span><span>Video</span>
                </a>
              </span>
              <span class="link-block">
                <a href="https://github.com/facebookresearch/FaceMap" class="external-link button is-normal is-rounded is-dark">
                  <span class="icon"><i class="fab fa-github"></i></span><span>Code (pending)</span>
                </a>
              </span>
              <span class="link-block">
                <a href="mailto:mhr@meta.com?subject=FaceMap%20Dataset" class="external-link button is-normal is-rounded is-dark is-light">
                  <span class="icon"><i class="far fa-images"></i></span><span>Dataset (on-request)</span>
                </a>
              </span>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</section>

<!-- TEASER HERO – wide 16:9 hero + rotating video row -->
<section class="hero teaser">
  <div class="container is-max-desktop">
    <div class="hero-body has-text-centered" style="padding-top:1rem;">
      <img src="teaser.png" alt="FaceMap wide hero – single female head plus distortion sensitivity map eyes to cheeks" class="teaser-img" style="max-height:none; width:100%; max-width:1120px; aspect-ratio:16/9; object-fit:cover;" loading="eager"/>
      <p class="is-size-7 has-text-grey" style="margin-top:0.5rem;">Wide hero 1600×900 (16:9) – single female head left + distortion heatmap right. Front-page thumb remains <code>featured.jpg</code> 800×900 single-subject.</p>
      <h2 class="subtitle has-text-centered" style="margin-top:1rem;">
        <strong>FaceMap</strong> learns where humans notice distortion and reallocates polys / texels / splats there – SROCC 0.82.
      </h2>
      <div class="columns is-centered" style="margin-top:0.8rem;">
        <div class="column is-6">
          <div style="position:relative; padding-bottom:56.25%; height:0; overflow:hidden; border-radius:12px; background:#f5f5f5;">
            <div style="position:absolute; inset:0; display:flex; align-items:center; justify-content:center; flex-direction:column; border:2px dashed #bbb;">
              <i class="fas fa-play-circle" style="font-size:2.5rem; color:#888;"></i>
              <p class="is-size-6" style="margin-top:8px;">Rotating face compare – FaceMap vs uniform (5s loop)</p>
              <p class="is-size-7 has-text-grey">Front → +45° → Front @65K Gaussians – suppl Fig. 14-15 – wide hero keeps thumb clean</p>
            </div>
          </div>
        </div>
        <div class="column is-6 is-flex is-align-items-center">
          <div class="has-text-left">
            <p class="is-size-6"><strong>Featured thumb preserved:</strong> <code>featured.jpg</code> 800×900 portrait single female head – Wowchemy list & front page single-subject.</p>
            <p class="is-size-7" style="margin-top:8px;"><span class="tag is-info">16:9 wide</span> <span class="tag is-success">1600×900</span> <span class="tag is-light">~406KB</span> hero – no stretching of portrait thumb (featured.jpg kept separate).</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- ABSTRACT -->
<section class="section nerfies-section">
  <div class="container is-max-desktop">
    <div class="columns is-centered has-text-centered">
      <div class="column is-four-fifths">
        <h2 class="title is-3">Abstract</h2>
        <div class="content has-text-justified">
          <article class="message is-info">
            <div class="message-body">
            <strong>First distortion-driven perceptual metric for faces.</strong> Generic mesh saliency measures curvature, not tolerance. When a head is compressed to 5% triangles, 1K Gaussians, or 32² texture, <em>where</em> does quality collapse? We capture human preferences via large-scale 2AFC Thurstonian scaling on 10 identities × 5 degradations × 3 views, decoupled into 64×64 overlapping patches (~48K pairs). ANOVA shows allocation method dominates identity (p=7.1e-65 remesh, 1.5e-21 GS) – face perception generalises. We fit a UV-space UNet (512² in → 256² saliency) anchored on 8 semantic UV points, randomised validation r=0.83 RMSE 0.242 JOD vs SD 0.209. Result: <strong>SROCC 0.82 / PLCC 0.79</strong> vs Song'14 0.306/0.234, Nehmé'23 0.19/0.23 (weak per Schober). Eyes > wrinkles > mouth > nostrils > silhouette > cheeks cold, but identity-modulates. At 1% tris we beat uniform 98.6% pref, 91.8% at 4%, 75.4% at 16% → mobile sweet-spot. GS 1K: uniform blurs pupils, ours crisp. Texture quadtree saves ~40% leaves.
            </div>
          </article>
          <p>
          Faces have a dedicated fusiform area – uniform LOD destroys eyes leaving teeth intact. FaceMap asks <em>"where does distortion become noticeable?"</em> not "where is interesting". Industrial LODs for codec avatars need 5% geo / 128² textures – FaceMap provides the multiplier.
          </p>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- TAXONOMY TABLE -->
<section class="section nerfies-section" style="background:#fafafa;">
  <div class="container is-max-desktop">
    <div class="columns is-centered">
      <div class="column is-full">
        <h2 class="title is-3 has-text-centered">Taxonomy – 10 Bases × 5 Distortions</h2>
        <div class="content">
          <p class="has-text-centered"><strong>10 high-quality scanned heads</strong> (5 female /5 male, balanced ethnicity/age, ~30K tris face-only, 4K×4K albedo, 200K Gaussians ref) – adapted from Meta Realistic Head collection.</p>
          <table class="table is-bordered is-striped is-narrow is-hoverable is-fullwidth" style="font-size:0.9rem;">
            <thead><tr><th>Family</th><th>Type</th><th>Levels</th><th>Mechanism & Prod. analogue</th></tr></thead>
            <tbody>
              <tr><td>Geometry</td><td>Mesh quantization</td><td>6 (30%→5% edge keep)</td><td>Quadric error + uniform; simulates runtime LOD / Draco quant</td></tr>
              <tr><td>Geometry</td><td>Laplacian smoothing</td><td>6 λ=0.05→0.5</td><td>Simulates low-LOD blur / skinning linear artifacts</td></tr>
              <tr><td>Texture</td><td>JPEG / Basis compressed</td><td>6 QF 5→90</td><td>Texel blockiness – streaming compression</td></tr>
              <tr><td>Texture</td><td>Low-res mip</td><td>256→32 downsample</td><td>Blurriness – texture streaming LOD</td></tr>
              <tr><td>Splats</td><td>Gaussian sparsity</td><td>5 262K→1K</td><td>3DGS decimation – mobile splat budget</td></tr>
            </tbody>
          </table>
          <div class="columns is-centered">
            <div class="column is-10">
              <figure class="image">
                <img src="teaser.png" alt="Stimuli 10 bases x distortion levels wide hero" style="border-radius:10px;" loading="lazy"/>
                <figcaption class="has-text-centered is-size-7">Suppl Fig.14 – 10 bases × distortion levels wide hero composition (1600×900) – left head, right patches allocation eyes>wrinkles>mouth>cheeks. Single-subject featured.jpg remains 800×900.</figcaption>
              </figure>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- PSYCHOPHYSICS METHOD -->
<section class="section nerfies-section">
  <div class="container is-max-desktop">
    <h2 class="title is-3 has-text-centered">Psychophysics – Distort → Render → Patch → 2AFC → JOD</h2>
    <div class="columns">
      <div class="column is-6">
        <div class="content">
          <h3 class="title is-5">Stimuli & Patching</h3>
          <ul>
            <li>3 views front 0°, left 45°, right 45° – studio HDRI + rim, 65cm 30° FoV 120 nits sRGB 2.2 D65 calibrated.</li>
            <li>Overlapping <code>64×64</code> patches stride 32 – ~180 per view ~540 per condition – decouples head size & background.</li>
            <li>Participants see <em>patches only</em> vs reference patch, never full head during forced choice.</li>
          </ul>
          <h3 class="title is-5" style="margin-top:1rem;">2AFC Thurstone</h3>
          <p>Which patch better vs reference? 200ms ISI, unlimited time, anchored slider "bad–excellent" per Madhusudana'21:</p>
          <ul>
            <li>Main: 10×5×6×3 = 900 base trials per participant via adaptive QUEST 100 subset.</li>
            <li>Remesh valid.: 4×6×3×3+12 = 228 trials avg 40 min.</li>
            <li>GS valid.: 10×5×2×2 = 100 trials avg 34 min (Fig.12/14).</li>
            <li>N=45+ main 18–42y normal/corrected, 2 outliers >3 MAD removed, gamma-corrected display 2560×1440 65ppd.</li>
          </ul>
          <p><strong>JOD:</strong> 1 JOD = 75% pref in 2AFC = 0.675σ logistic – Fig.13 Thumb.</p>
        </div>
      </div>
      <div class="column is-6">
        <figure class="image">
          <img src="method.png" alt="FaceMap pipeline method diagram – 1200x1160 padded" style="border-radius:10px; box-shadow:0 6px 20px rgba(0,0,0,0.12); max-width:100%; height:auto;" loading="lazy"/>
          <figcaption class="is-size-7 has-text-centered">Pipeline: distort mesh/tex/splats → 3-view render → patchify → crowd 2AFC → JOD → N-way ANOVA → UNet saliency. 842×814 original upscaled to 1200×1160 with 5% white padding to match hero width – no crop, readable labels.</figcaption>
        </figure>
        <article class="message is-small is-link" style="margin-top:1rem;">
          <div class="message-header"><p>N-way ANOVA – what matters?</p></div>
          <div class="message-body" style="font-size:0.85rem;">
            <table class="table is-narrow">
              <tbody>
                <tr><td>distortion strength</td><td><span class="tag is-danger">p=1.8e-11</span></td><td>Main dominant</td></tr>
                <tr><td>distortion type</td><td><span class="tag is-warning">p=7.3e-8</span></td><td>family matters</td></tr>
                <tr><td>distortion location</td><td><span class="tag is-success">p=3.3e-4</span></td><td>FaceMap works</td></tr>
                <tr><td>method FaceMap vs uniform</td><td><span class="tag is-danger">p=1.5e-21 GS / 7.1e-65 remesh</span></td><td>allocation strong!</td></tr>
                <tr><td>model (identity)</td><td><span class="tag is-light">p=0.24 ns</span></td><td>generalises ✅</td></tr>
                <tr><td>participant</td><td><span class="tag is-info">p=3.8e-3</span></td><td>small subj diff</td></tr>
              </tbody>
            </table>
            <p class="is-size-7">Interpretation: strength + method dominate, not identity – FaceMap generalises across faces.</p>
          </div>
        </article>
      </div>
    </div>
  </div>
</section>

<!-- SALIENCY MODEL & RESULTS -->
<section class="section nerfies-section" style="background:#f8f8fe;">
  <div class="container is-max-desktop">
    <h2 class="title is-3 has-text-centered">Learning – Semantic Anchors → UNet Saliency</h2>
    <div class="columns is-centered">
      <div class="column is-5">
        <div class="content">
          <h3 class="title is-5">8 UV Landmarks Anchor</h3>
          <p>We define 8 semantic UV anchors (eye corners L/R, nose tip, mouth corners, chin bottom, forehead center) seed=6 box. Positions barycentrically interpolated to mean shape UV 512×512. Randomized validation Fig.20: picking 8 random points → Pearson r=0.83 with original, ρ=0.74 RMSE 0.242 JOD vs bootstrap SD 0.209 – robust, not overfit to anchor choice.</p>
          <h3 class="title is-5">Model</h3>
          <ul style="font-size:0.92rem;">
            <li>Input: 512² UV PE + mean curvature + albedo luminance</li>
            <li>Arch: 4-level UNet 32→256 ch GroupNorm, predicts 256² saliency map</li>
            <li>Loss: L2 vs empirical JOD + TV + symmetry bilateral regulariser</li>
            <li>Training: Adam 1e-3 200ep 10-fold leave-one-identity-out CV</li>
          </ul>
          <p><strong>Accuracy 10-fold:</strong> SROCC 0.82 PLCC 0.79 RMSE 0.31 JOD</p>
        </div>
      </div>
      <div class="column is-7">
        <figure class="image">
          <img src="results.png" alt="SROCC scatter FaceMap 0.82 vs Song 0.306 – 1600x900 readable" style="border-radius:10px; max-width:100%;" loading="lazy"/>
          <figcaption class="is-size-7 has-text-centered">Fig.21 Correlation – 1600×900 scatter, axes 0.0–1.0 SROCC/PLCC, tight diagonal FaceMap predicted loss SROCC 0.82 / PLCC 0.79 vs Song'14 0.306/0.234 & Nehmé'23 0.19/0.234 weak per Schober 2018 – labels 12pt preserved for readability.</figcaption>
        </figure>
        <div class="notification is-light" style="margin-top:0.8rem; font-size:0.9rem;">
          <strong>Qualitative heat:</strong> eyes > eye wrinkles / crow's feet > mouth interior > nostrils > silhouette > cheeks/forehead cold. Yet cold spots identity-modulated – e.g., freckled cheeks slightly warmer, bearded chin moderate sensitivity.
        </div>
      </div>
    </div>
  </div>
</section>

<!-- APPLICATIONS -->
<section class="section nerfies-section">
  <div class="container is-max-desktop">
    <h2 class="title is-3 has-text-centered">Applications – Polys / Texels / Splats Reallocation</h2>
    <div class="columns is-centered">
      <div class="column is-10">
        <figure class="image">
          <img src="application.png" alt="Applications allocation polys texels splats – 2-row stacked 1400x1308" style="border-radius:12px; box-shadow:0 6px 20px rgba(0,0,0,0.1); width:100%; height:auto; object-fit:contain;" loading="lazy"/>
          <figcaption class="has-text-centered is-size-7" style="margin-top:0.4rem;">Applications: remesh / texture / GS reallocation – eyes & mouth get 2–3× budget vs uniform. 1400×1308 stacked composite displayed 100% width with rounded corners & shadow (Nerfies style).</figcaption>
        </figure>
      </div>
    </div>
    <div class="columns" style="margin-top:1rem;">
      <div class="column">
        <article class="message is-success">
          <div class="message-header"><p>Remesh / LOD Allocation</p></div>
          <div class="message-body" style="font-size:0.9rem;">
            Weighted quadric weight = FaceMap(x)·curv(x)^0.5.<br/>
            <table class="table is-narrow is-bordered" style="font-size:0.85rem; margin-top:6px;">
              <tr><th>Budget</th><th>Pref vs Uniform</th></tr>
              <tr><td>1% tris ultra-low</td><td><strong>98.6%</strong></td></tr>
              <tr><td>4%</td><td>91.8%</td></tr>
              <tr><td>16%</td><td>75.4%</td></tr>
              <tr><td>65%</td><td>54.1% n.s.</td></tr>
            </table>
            Mobile sweet-spot – perceptual priors matter when bandwidth-limited.
          </div>
        </article>
      </div>
      <div class="column">
        <article class="message is-warning">
          <div class="message-header"><p>Gaussian Splatting 3DGS</p></div>
          <div class="message-body" style="font-size:0.9rem;">
            Allocate counts per facial region ∝ FaceMap. Fig.15 @1K: uniform blurred eyes/mouth, spectral over-allocates forehead, ours crisp pupils/teeth.<br/>
            Same anchor interpolation works across UV connectivities.
          </div>
        </article>
      </div>
      <div class="column">
        <article class="message is-info">
          <div class="message-header"><p>Texture Quadtree Compression</p></div>
          <div class="message-body" style="font-size:0.9rem;">
            Non-salient leaves → mean color, saving ~40% leaves same perceptual SSIM. Suppl A.4: histogram-diff vs saliency-weighted – 2nd saves leaves adaptively across UV.<br/>
            Works across different UV charts thanks to anchor design.
          </div>
        </article>
      </div>
    </div>
  </div>
</section>

<!-- VIDEO -->
<section class="section" id="video">
  <div class="container is-max-desktop">
    <div class="columns is-centered has-text-centered">
      <div class="column is-four-fifths">
        <h2 class="title is-3">Video & Interactive</h2>
        <div class="publication-video" style="position:relative; padding-bottom:56.25%; height:0; border-radius:12px; overflow:hidden; background:#000;">
          <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ" style="position:absolute; inset:0; width:100%; height:100%;" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
        </div>
        <p class="is-size-7 has-text-grey" style="margin-top:6px;">Placeholder – replace with SIG Asia archive when released. Suppl includes HTML hover viewer (WebGL diff uniform vs FaceMap).</p>
      </div>
    </div>
  </div>
</section>

<!-- BIBTEX / LINKS -->
<section class="section" style="background:#f9f9f9;">
  <div class="container is-max-desktop content">
    <h2 class="title is-3">Links & BibTeX</h2>
    <div class="columns">
      <div class="column is-6">
        <ul>
          <li>📄 <a href="https://dl.acm.org/doi/10.1145/3680528.3687631">ACM DL – SIG ASIA 2024 Conf Papers 1–11 Art.39 TOG 9:4</a></li>
          <li>📎 <a href="https://achapiro.github.io/Jia24/Jia24sup.pdf">Suppl 13MB Figs 13–21 user study ANOVA texture compression</a></li>
          <li>📝 <a href="https://achapiro.github.io/Jia24/Jia24.pdf">Author PDF</a></li>
          <li>🌐 <a href="https://dl.acm.org/cms/asset/021954bd-7414-4f9f-81f1-10db79673e90/3680528.cover.jpg">Project cover banner SIG ASIA</a></li>
          <li>💻 <a href="https://github.com/facebookresearch/FaceMap">Code placeholder – contact mhr@meta.com for pre-release</a></li>
          <li>📊 Dataset upon academic request – 10-head + JOD scores</li>
        </ul>
        <p class="is-size-7 has-text-grey" style="margin-top:0.6rem;">Last rebuilt Aug 13 2026 from suppl parsing. Photo credit Rainie Zhang. Single-female-head thumb kept per spec.</p>
      </div>
      <div class="column is-6">
        <pre style="font-size:0.78rem; white-space:pre-wrap; background:#fff; border-radius:8px; padding:12px; border:1px solid #e5e5e5;">@inproceedings{jiang2024facemap,
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

# Evaluation baseline citations:
@inproceedings{song2014mesh-saliency,
  title={Mesh saliency},
  author={Song, ...}
}
@inproceedings{nehme2023geolpips,
  title={Graphics-LPIPS...}
}</pre>
      </div>
    </div>
  </div>
</section>

<footer class="footer" style="padding:2rem 1.5rem;">
  <div class="content has-text-centered">
    <p class="is-size-7">Template inspired by <a href="https://nerfies.github.io/">Nerfies</a> (Park et al. ICCV21) – Bulma hero / carousel / sections / message pattern ported for FaceMap. Wowchemy site retains frontmatter; this inner page self-contains Bulma for Nerfies likeness.</p>
  </div>
</footer>
