
Separate deterning coverage (the stencil step) from rendering the pattern (the cover step).
Typically used in conjunction with [[Multi-sampling]].

# Principle

## The stencil step

Shapes are rendered into the stencil buffer, typically using triangle fans, counting the number of times each pixel is touched.

![[stencil-cover.gif]]
([Image source](https://medium.com/@evanwallace/easy-scalable-text-rendering-on-the-gpu-c3f4d782c5ac))

## The cover step

# Optimizations

- To render bézier curves in the stencil buffer, use a pivot point for the coarse version of the path (including the bezier hull), then for each bezier curve have a local pivot point to carve the curve out of the hull (for example pick the first endpoint).
- Non-overlapping shapes can be batched ([[Skia graphite]] does that).

# Limitations

- Cannot render multiple overlapping shapes at the same time.
- Very memory-intensive
- Need to switch back and forth between the stencil buffer and main render target which tends to be inefficient both in terms of driver overhead 