---
layout: ../../layouts/RawLayout.astro
title: "refactor shared base styles"
status: "pending"
focus: "extract common CSS from RawLayout and DetailLayout into src/styles/global.css"
next: "replace duplicated style blocks with a single import in both layouts"
order: 50
---

# refactor shared base styles

RawLayout and DetailLayout currently duplicate the full set of base styles (tokens, typography, links, code blocks, media). Extract into a shared `src/styles/global.css` and import in both layouts without changing visual output.
