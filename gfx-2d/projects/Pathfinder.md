#tiling

Source code: https://github.com/pcwalton/pathfinder

Several approaches were tried in pathfinder:

# PF2

Tessellation approach based on a trapezoidal partition. Each trapezoid generates a bounding rectangle and an "internal" rectangle: the latter is used to pre-fill opaque areas and use the z-buffer to reduce overdraw while the latter contains the equation of the edges on both sides of the trapezoid and renders the curve with anti-aliasing. The approach works generally well but has a few glitches in almost vertical edges (would be fixable with conservative rasterization).

# PF2(text)
An approach similar to stencil-and-cover except that it renders the winding number in a float texture with anti-aliasing. Very fast for rather small text but performance degrades quickly with very high resolution so a good solution for text but not great for SVG paths. No overdraw mitigation.

# PF3

## CPU binning version
 
Same approach as pf2-text on the GPU but large paths are split into 16x16 tiles which are rasterized independently. A sweep-line algorithm is used to generate the tiles and the algorithm is somewhat similar to a software rasterizer but writes the edges that affect each tile rather than filling pixels. Full tiles are used to discard content from other paths below and reduce overdraw.
  - Explained here: https://nical.github.io/posts/a-look-at-pathfinder.html
  - Promising approach in that it is less of a numerical stability nightmare than many other approaches, is rather simple to implement and doesn't rely on advanced GPU features.
  - Performance depends on how fast the tiling algorithm is implemented and there's a trade-off between CPU and GPU cost depending on the tile size.
  - The sweep-line algorithm was later replaced by the tile decomposition algorithm from the RAVG paper.
  - A GPU tiling algorithm was later added in addition to the CPU tiling
  - Also has a compute code path that works like piet-metal

## Compute version

- Binning done by [a compute shader](https://github.com/servo/pathfinder/blob/21ec6fa933547636bc6c5ee8f0dd4a0ea3fcd062/shaders/d3d11/bin.cs.glsl#L188), the code is actually very similar to the [CPU version](https://github.com/servo/pathfinder/blob/21ec6fa933547636bc6c5ee8f0dd4a0ea3fcd062/renderer/src/tiler.rs#L191).
- Edge list is stored as a linked list in a storage buffer.
