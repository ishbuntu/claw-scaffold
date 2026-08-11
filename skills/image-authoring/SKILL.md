---
name: image-authoring
description: Create images by authoring them as code (SVG, matplotlib, PIL). Use when asked to generate/draw/illustrate and no image-generation model is available.
triggers: image, draw, illustrate, logo, icon, poster, banner, diagram, wallpaper
---

# Image authoring

There is no text-to-image model here. You ARE the artist: author the image
as code and render it to a file.

1. Pick the medium: SVG for logos/icons/flat art (hand-write the XML),
   matplotlib for charts/data art, PIL for raster composition.
2. Write the code, render to `{WORKSPACE}/images/{slug}.png` (or `.svg`),
   and show the result by absolute path.
3. Iterate on feedback by editing the code — keep the source next to the
   output so every image stays regenerable.

For photorealism requests: say plainly the output will be stylized vector
or procedural art, and offer the closest achievable style. Never present
the result as model-generated imagery.
