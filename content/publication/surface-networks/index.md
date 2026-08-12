+++
title = "Surface Networks"
date = 2018-03-28T20:04:23-04:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Ilya Kostrikov", "**Zhongshi Jiang**", "Daniele Panozzo", "Denis Zorin", "Joan Bruna"]

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
publication = "IEEE Computer Society Conference on Computer Vision and Pattern Recognition (Oral Presentation)"
publication_short = "*IEEE CVPR*, 2018 (*Oral Presentation*)"

# Abstract and optional shortened version.
abstract = "We study data-driven representations for three-dimensional triangle meshes, which are one of the prevalent objects used to represent 3D geometry. Recent works have developed models that exploit the intrinsic geometry of manifolds and graphs, namely the Graph Neural Networks (GNNs) and its spectral variants, which learn from the local metric tensor via the Laplacian operator. Despite offering excellent sample complexity and built-in invariances, intrinsic geometry alone is invariant to isometric deformations, making it unsuitable for many applications. To overcome this limitation, we propose several upgrades to GNNs to leverage extrinsic differential geometry properties of three-dimensional surfaces, increasing its modeling power. In particular, we propose to exploit the Dirac operator, whose spectrum detects principal curvature directions --- this is in stark contrast with the classical Laplace operator, which directly measures mean curvature. We coin the resulting model the Surface Network (SN). We demonstrate the efficiency and versatility of SNs on two challenging tasks: temporal prediction of mesh deformations under non-linear dynamics and generative models using a variational autoencoder framework with encoders/decoders given by SNs."
summary = "An adapation of graph neural networks on triangle meshes, taking advantage of discrete differential operators like Laplacians and Diracs."

# Featured image thumbnail (optional)
image_preview = "SurfaceNetworks.png"

# Is this a selected publication? (true/false)
selected = false

# Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter the filename (excluding '.md') of your project file in `content/project/`.
#   E.g. `projects = ["deep-learning"]` references `content/project/deep-learning.md`.
projects = []

# Tags (optional).
#   Set `tags = []` for no tags, or use the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["Oral Presentation"]

# Links (optional).
url_pdf = "files/SurfaceNetworks.pdf"
url_preprint = "https://arxiv.org/pdf/1705.10819.pdf"
url_code = "https://github.com/jiangzhongshi/SurfaceNetworks"
url_dataset = "https://github.com/jiangzhongshi/SurfaceNetworks#data"
url_project = ""
url_slides = ""
url_video = ""
url_poster = ""
url_source = ""

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form `[{...}, {...}, {...}]`.
url_custom = [{name = "Supplemental", url = "https://cims.nyu.edu/gcl/papers/2018-Surface-Networks-Supplemental.pdf"}]

# Does this page contain LaTeX math? (true/false)
math = false

# Does this page require source code highlighting? (true/false)
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++
