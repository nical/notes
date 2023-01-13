
# Sweep-line algorithms 

See also: [[Sweep-line algorithm]].

General idea:
-   Sort input edges from top to bottom.
-   Scan the output area, maintaining a list of edges that intersect with the current scanline (the active edges). The active edge list is sorted from left to right
-   When the scanline moves down, the active edges that no-longer intersect it are removed, while some new edges will be added. Since the input edge list is sorted we simply have to look at when the next edge starts.
-   The scanline is traversed from left to right. At each step the winding number is updated. This winding number, combined with the winding rule, defines what is covered and what is not.
  

Most path renderers rely on of the properties are sweep line algorithms:
-   Large amount of memory bandwidth (reads and writes) is associated with the output image so it makes sense for the algorithm to output content in a way that will allow cache-efficient interaction with the output. The y-sweep has the nice property that it follows the memory layout of the output image.
-   In order to decide what is in and out of the path, the algorithm has to consider edges. It would be very expensive to visit all edges at each step. Thankfully we don't need to visit all of them. Only the edges between the point in consideration and some point that is known to be outside of the shape. Again, horizontal scanlines help a lot here:
-   It provides a framework for reducing the number of edges that have to be considered by the algorithm (only edges intersecting the horizontal line at the current y position of the algorithm).
-   The work done while scanning a line (for example from left to right) is very incremental, in other words the next pixels will benefit from a lot of the work done on the same scanline.
-   Note that some algorithms exploit these properties in different ways. Font-rs in particular is set apart from most of the others. I'll get back to that.

# Curves

Most of the implementations I've looked at internally approximate curves with sequences of line segments (also referred to as curve flattening). There are different algorithms to achieve that, like:

-   [https://raphlinus.github.io/graphics/curves/2019/12/23/flatten-quadbez.html](https://raphlinus.github.io/graphics/curves/2019/12/23/flatten-quadbez.html)
-   More commonly using forward difference (skia, etc.).

The naïve approach would be to flatten all edges in a first pass before running the sweep-line algorithm on only line segments. However it is more efficient to scan curves and calculate/update the approximation at the intersection between the sweep line and each active edge. For example an active edge in skia contains the curve itself as well as the linear approximation at the intersection between the curve and the sweep line and any additional state required to update it. When the sweep line moves, each curve updates its approximation.

There is an important tradeoff between precision and performance. Some flattening approaches are pretty expensive. The forward difference used in skia wouldn't work for every use cases. can't keep track of the curve parameter `t` along the approximation (skia doesn need it), it doesn't exactly converge to the end of the curve, which isn't an issue for that algorithm as long as there is no discontinuity in y. On the other hand it relies on simple and cheap instructions while other approaches require a lot of divisions and square roots. It can also afford to generate more line segments than strictly necessary since it doesn't come at the cost of storing or processing more segments.

TODO: more details about the approach.

Also worth noting that skia does most of this math using fixed-point numbers instead of floats. I don't know how important this is on a modern CPU

# Prefix sum

As hinted earlier, parts of some rasterization algorithms can be thought of as [[Prefix sum]]s. Thinking in terms of prefix sums can help see properties of some of the problem and some of the algorithms.

-   Winding number at any pixel is the winding number at its left neighbor + contribution of edges between the two pixels.
-   Fast prefix sum algorithms are all about minimizing redundant work.
-   Unfortunately it's also very sequential by nature. It is hard to efficiently rasterize a path using multiple threads, at least at the pixel level. However there is some literature on parallel prefix sums that can be used as inspiration here as well.

# Anti-aliasing

-   Super-sampling
-   Area coverage
  

TODO: conflation artifacts

# A look at various approaches

## Skia and friends

Skia had a multisampling rasterizer which was replaced with one using analytic aa
The following code was ported/inspired by skia's multisampling rasterizer:
-   [https://github.com/jrmuizel/raqote/blob/master/src/rasterizer.rs](https://github.com/jrmuizel/raqote/blob/master/src/rasterizer.rs)
-   [https://github.com/RazrFalcon/tiny-skia/blob/afa3172310bb7a30e44911ef825f06dd900a4d85/src/edge.rs](https://github.com/RazrFalcon/tiny-skia/blob/afa3172310bb7a30e44911ef825f06dd900a4d85/src/edge.rs)

This presentation explains some of what goes into the analytic aa one: [https://skia.org/docs/dev/design/aaa/](https://skia.org/docs/dev/design/aaa/)

## Font-rs

[https://medium.com/@raphlinus/inside-the-fastest-font-renderer-in-the-world-75ae5270c445](https://medium.com/@raphlinus/inside-the-fastest-font-renderer-in-the-world-75ae5270c445)

Instead of processing edges in a swee-line fashion, goes through all edges in their original order and rasterizes them in a temporary accumulation buffer. The output of this rasterization stage is the signed coverage delta at each pixel. Conceptually each pixel of the accumulation buffer represents the difference between the winding number of this pixel and the previous one. So it steps things up for running a simple prefix sum to determine the winding number.

The important ideas here are:

-   Instead of using a sparse intermediate representation (scanlines and active edges), font-rs uses a dense representation (an accumulation buffer the size of the output.
-   The sparse approaches have the advantage of not touching pixels that the path do not cover, while font-rs has to run a prefix sum over all pixels of the accumulation buffer.
-   On the other hand, the prefix sum is a very tight and efficient loop with little to no branching and amenable to simd instructions.
-   Tradeoff that depends on the type of content being rendered. Font-rs works very well for small resolutions (typical when rendering glyphs).


There are also some hybrid approaches where coverage deltas are built from the input edges but instead of being rendered in a dense accumulation buffer, they are built into a sparse data structure (for example a vector which is then sorted). This is what gouache and other work from the same author do. It also looks very similar to what spinel does (GPU path renderer), although I'm not sure that the internal representation is actually coverage deltas (it is at least something that provides a similar type of information).

## libart

https://web.archive.org/web/20191017233828/https://people.gnome.org/~mathieu/libart/internals.html

## Freetype

TODO