
---
Authors: Raph Levien
---

# Overview

Source code: [https://github.com/google/font-rs](https://github.com/google/font-rs)
Blog post: https://medium.com/@raphlinus/inside-the-fastest-font-renderer-in-the-world-75ae5270c445

A glyph rasterizer running on the CPU.

Runs in two steps:
 - delta-coverage rasterization: edges are rasterized in a temporary floating point buffer (the accumulation buffer) which, for each pixel, represents the contribution of the outline.
 - Scan: n a second step, the accumulation buffer is "integrated": A [[Prefix sum]] is done over the accumulation buffer, resulting into a winding number per pixel on which is applied the fill rule, giving the final pixel alpha value which is written in the destination mask.

## Delta coverage rasterization

All texels in the accumulation buffer that don't intersect an outline are equal to 0 and texels that touch the outline have a positive or negative value depending on the direction of the outline.

![](https://miro.medium.com/max/1091/0*eK0MPlbHURlFQ1ng.)

See https://github.com/raphlinus/font-rs/blob/297f749066f874a8e4b63e3824d9b4a79a6f7b69/src/raster.rs#L44

## Scan

All pixels with a non-zero integrated value are in the shape. This is where much of the time is spent. The intergation is done by a very tight branchless loop optimized with SIMD SSE instructions on x86.

See https://github.com/raphlinus/font-rs/blob/297f749066f874a8e4b63e3824d9b4a79a6f7b69/src/accumulate.rs#L30

# Notes

## Sparse versus dense representation

Font-rs is a good illustration of the "simpler is faster" rule of thumb. Font-rs differs with freetype and many other font rasterizers in that the latters use *sparse* representations for their intermediate representations (sorted edge lists for a [[Sweep-line algorithm]]) while font-rs has a *dense* representation (the accumulation buffer).

The cost of the image representation of the accumulation buffer goes up (both in processing time and memory usage) with the resolution of the output image. Fonts being typically very small, this approach works very well. For very large vector graphics, this approach has some interesting challenges (especially when ported to the gpu). I suspect that to get good performance out of this, one would want to batch several paths in the same accumulation buffer atlas to avoid alternating between the accumulation and integration phases.

## Parallelism

Because font-rs is typically useful for rasterizing small shapes (glyphs), there isn't a lot to gain from multi-threading since the number of edges and pixels are typically small. That said it is a very good fit for [[SIMD]].

Since glyphs are typically numerous and independent, it is trivial to rasterize a number of them in parallel, not very much to say about that.

If it were applied for larger and more complex shapes, it should be noted that font-rs avoids steps that are typically tedious to perform in parallel, such as sorting edges and doing a sweep line.
Concurrent writes into the accumulation buffer would be challenging to make fast. One could try to deffer writing into the accumulation buffer by pushing into thread-local vectors of `{ x, y, coverage }` and then either have a single thread write them into the accumulation buffer or even skip the accumulation buffer and use a sorted version of these vectors for the final scan (that sounds very similar to the pixel-segment concept in [[Spinel]]/[[Forma]]).

The scan phase can trivially do each row in parallel, provided there is evenough pixels for it to be worth it.

# TODO merge previous note

Turns out I had several nots about font-rs, merge the content below with what's above.

Instead of processing edges in a swee-line fashion, goes through all edges in their original order and rasterizes them in a temporary accumulation buffer. The output of this rasterization stage is the signed coverage delta at each pixel. Conceptually each pixel of the accumulation buffer represents the difference between the winding number of this pixel and the previous one. So it steps things up for running a simple prefix sum to determine the winding number.

The important ideas here are:

-   Instead of using a sparse intermediate representation (scanlines and active edges), font-rs uses a dense representation (an accumulation buffer the size of the output.
-   The sparse approaches have the advantage of not touching pixels that the path do not cover, while font-rs has to run a prefix sum over all pixels of the accumulation buffer.
-   On the other hand, the prefix sum is a very tight and efficient loop with little to no branching and amenable to simd instructions.
-   Tradeoff that depends on the type of content being rendered. Font-rs works very well for small resolutions (typical when rendering glyphs).


There are also some hybrid approaches where coverage deltas are built from the input edges but instead of being rendered in a dense accumulation buffer, they are built into a sparse data structure (for example a vector which is then sorted). This is what gouache and other work from the same author do. It also looks very similar to what spinel does (GPU path renderer), although I'm not sure that the internal representation is actually coverage deltas (it is at least something that provides a similar type of information).
