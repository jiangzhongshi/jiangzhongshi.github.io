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
image_preview = "PE-2019.png"

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
math = false

# Does this page require source code highlighting? (true/false)
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
#[header]
#image = "PE-2019.png"
#caption = "captions is here"

#image = []
+++
