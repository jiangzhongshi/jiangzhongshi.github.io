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
summary = "A robust and efficient way optimize the distortion of bijective maps. Applicable to 2D (bijective surface parameterization) as well as 3D (intersection free deformation) problems."

# Featured image thumbnail (optional)
image_preview = "SCAF-2017.png"

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
url_pdf = "https://par.nsf.gov/servlets/purl/10047009"
url_preprint = ""
url_code = "https://github.com/jiangzhongshi/Scaffold-Map"
url_dataset = ""
url_project = ""
url_slides = "files/SCAF_talk.pdf"
url_video = ""
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
[header]
image = "SCAF-2017.png"
caption = "captions is here"

+++
