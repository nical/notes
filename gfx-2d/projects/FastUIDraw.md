
Link: https://github.com/intel/fastuidraw

* [[Tessellation]]
* Resolution-dependent tessellation
* Relies a lot on caching geometry.
* Splits all images into fixed size tiles and store them into a big atlas with padding to avoid sampling artifacts
  * Very similar to a sparse virtual texture
  * Using an indirection texture
  * Optimizes when several tiles are filled with the same color into one tile.
* Uses the depth buffer to clip out (rendering the clip path before)
  * Clips are not anti-aliased
* Separate solution for glyphs

Some technical details in these blog posts:
- https://web.archive.org/web/20190116182936/https://01.org/fast-ui-draw/blogs/krogovin/2016/fast-ui-draw-technical-details-1
- https://web.archive.org/web/20190116182939/https://01.org/fast-ui-draw/blogs/krogovin/2016/fast-ui-draw-technical-details-2.


# Overview

Canvas-like immediate-mode API, focus on minimizing state changes and on working well on intel integrated GPUs (the lib itself is made by intel).

FastUIDraw uses the depth buffer for clipping (no anti-aliasing), but also writes into the depth buffer when rendering paths. I assume it is doing a front-to-back pass for opaque geometry with depth test and writes enabled to save bandwidth the like webrender does, otherwise there wouldn't be much point in writing paths to the depth buffer.

# Curves

FastUIDraw doesn't do any resolution-independent tessellation, curves are flattened before tessellation using a naive recursive mid-point split until the flattened approximation is within a certain tolerance threshold (can be either distance or curvature threshold). Relevant code in path.cpp.

# Fills

FastUIDraw fills paths on the GPU using tessellated geometry. The tessellator is a fork of the GLU tessellator.

https://github.com/01org/fastuidraw/tree/master/src/3rd_party/glu-tess

-   It uses a half edge data structure.
-   Resolution-dependent tessellation, with LOD cached in the path
-   Relies a lot on caching geometry.
-   Uber-shader (supposedly works well on intel)
-   The texture cache uses a sparse virtual texture for images, gradients and colors and a more traditional texture packing algorithm for glyphs.
-   Separate solution for glyphs (using signed distance fields)
-   Paths are recursively split before tessellation in order to avoid very large/expensive tessellation and perform culling (see fastuidraw/painter/filled_path.cpp look for SubsetPrivate)
-   Paths are tweaked before tessellation to mitigate precision issues (see CoordinateConverter in src/fastuidraw/painter/filled_path.cpp).
-   GLU's tessellator fails with edges that overlap
-   Single-contour polygons are first tested to see if they can be rendered with a simple triangle fan to avoid running the more expensive monotone polygon tessellator
-   The tessellator tries to reduce the amount of vertex data generated by producing a mix of triangle fans and strips

# Strokes

Tessellation again.
-   Tessellate strips of triangles along the stroke
-   Not sure how/if it handles self-intersecting transparent strokes #TODO. Maybe using the depth buffer?
-   Recursive data structure for culling like with fills
-   Vertex-aa for anti-aliasing

# Clipping

Clipping uses the depth buffer (no anti-aliasing).
-   To clip out, just write to the depth buffer with a higher z than the path to be clipped.
-   To clip in, keep track of four clip planes to do a cheap clip in rectangle (which is the bounding box of the clip path) in the shader and clip out the complement of the path within that rectangle using the depth buffer.
-   See: https://01.org/fast-ui-draw/blogs/krogovin/2016/fast-ui-draw-technical-details-2