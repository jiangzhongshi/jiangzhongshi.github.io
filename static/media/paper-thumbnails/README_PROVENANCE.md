# Paper Thumbnail Provenance

Images pulled 2026-08-12 for jiangzhongshi.github.io homepage refresh.

## Sources

- **Large-scale Codec Avatars (arXiv:2604.02320)**
  - `lca-method.png` = fig3-method.png from https://ar5iv.labs.arxiv.org/html/2604.02320/assets/fig3-method.png
  - `lca-sota.png` = fig_sota_comparisons.png from same ar5iv page
  - Page URL: https://ar5iv.labs.arxiv.org/html/2604.02320 , https://junxuan-li.github.io/lca (project linked in paper)
  - Stored as: `static/media/paper-thumbnails/lca-method.png`, `content/publication/large-scale-codec-avatars/featured.png`

- **FRESA (arXiv:2503.19207) - CVPR 2025**
  - `fresa-pipe.png` = fig_pipe_v4_c.png from https://ar5iv.labs.arxiv.org/html/2503.19207/assets/fig_pipe_v4_c.png
  - `fresa-four.png` = fig_four_c.png from same
  - Page URL: https://ar5iv.labs.arxiv.org/html/2503.19207
  - Stored as: `static/media/paper-thumbnails/fresa-pipe.png`, `content/publication/fresa/featured.png`

- **PhySkin (arXiv:2603.27013) - 2026**
  - `physkin-teaser.png` = finalteaser.png from https://ar5iv.labs.arxiv.org/html/2603.27013/assets/img/finalteaser.png
  - `physkin-arch.jpg` = architecture_v5_el.jpg from same host
  - Page URL: https://ar5iv.labs.arxiv.org/html/2603.27013
  - Stored as: `static/media/paper-thumbnails/physkin-teaser.png`, `content/publication/physkin/featured.png`

- **HyperBones (arXiv:2605.20460) - 2026**
  - `hyperbones-arch.png` = architecture_v3.png from https://ar5iv.labs.arxiv.org/html/2605.20460/assets/architecture_v3.png
  - `hyperbones-results.png` = results_ours.png from same
  - Page URL: https://ar5iv.labs.arxiv.org/html/2605.20460
  - Stored as: `static/media/paper-thumbnails/hyperbones-arch.png`, `content/publication/hyperbones/featured.png`

- **Latent Dynamics (arXiv:2605.21478)**
  - `latent-teaser.jpg` placeholder sample from ablation figure c05_0.jpg from https://ar5iv.labs.arxiv.org/html/2605.21478/assets/figs/ablation/c05_0.jpg
  - Page URL: https://ar5iv.labs.arxiv.org/html/2605.21478
  - Stored as: `content/publication/latent-dynamics/featured.jpg`

- **MHR (arXiv:2511.15586) - Momentum Human Rig**
  - `mhr-fig.png` = fig3_res.jpg from https://ar5iv.labs.arxiv.org/html/2511.15586/assets/figures/fig3_res.jpg (renamed png for Wowchemy compat)
  - Page URL: https://ar5iv.labs.arxiv.org/html/2511.15586
  - Stored as: `content/publication/mhr/featured.jpg`

- **Rethinking Video-Text (arXiv:2407.13094) - ECCV 2024**
  - `rethinking-teaser.png` = teaser_eg2.png from https://ar5iv.labs.arxiv.org/html/2407.13094/assets/figures/teaser_eg2.png
  - Page URL: https://ar5iv.labs.arxiv.org/html/2407.13094
  - Stored as: `content/publication/rethinking-video-text/featured.png`

- **ABC Dataset (CVPR 2019) - Big CAD**
  - `abc-dataset.jpg` from https://cims.nyu.edu/gcl/papers/2019-ABC-Dataset.jpg (NYU GCL official thumbnail, also visible on https://deep-geometry.github.io/abc-dataset/)
  - Page URLs: https://cims.nyu.edu/gcl/publications.html, https://deep-geometry.github.io/abc-dataset/
  - Stored as: `static/media/paper-thumbnails/abc-dataset.jpg`, `content/publication/abc-dataset/featured.jpg`

- **FaceMap (SIGGRAPH Asia 2024)**
  - Attempted clone of private repo `jiangzhongshi/SIGASIA23-FaceMap` via gh clone – private but authenticated; no public teaser image found. Image search via /opt/hatch/bin/image-search "FaceMap SIGGRAPH Asia 2024" returned no project-specific image (only generic facial maps). Left intentionally without featured image to avoid invention – can be added later from author PDF.
  - Provenance attempt: https://arxiv.org search blocked, no arXiv version found; SIGGRAPH Asia 2024 Conference Papers not open HTML.

- **Declarative Specification (SIGGRAPH Asia 2022)**
  - NYU page https://cims.nyu.edu/gcl/papers/2022-declarative-spec.png returned 404. ar5iv version not found (journal version ACM Trans Graphics). Project page may host teaser but not discovered. Left without image; prior publication folder remains without featured – will fallback to generic.

All downloads done with curl -L -s using HTTPS and final 2xx check. Thumbnails verified as image magic (PNG/JPEG). No phone numbers or private data included.

Notes:
- TheOrg mirror links removed earlier; admin profile now only references LinkedIn (//linkedin.com/in/zhongshi-jiang/) per user request replacing TheOrg.
- Wowchemy: featured.* in same folder as index.md is auto-picked as preview; static/media/paper-thumbnails centralized for reuse on homepage preview artifact.
