# Lyon (fill tessellator)

We vaguely refer to "the tessellator" in this document to talk about the algorithm that tessellate a Path into a fill operation. The tessellator's job is to output a sequence of triangles in a way that is practical for consumption by various GPU APIs such as OpenGL or Direct3D. The tessellator itself is self-contained and independent of the underlying GPU backend.

See also the Wikipedia page on the topic of [polygon triangulation](https://en.wikipedia.org/wiki/Polygon_triangulation) (we use triangulation and tessellation interchangeably).

## Y-monotone tessellation

The algorithm is based on the properties of [monotone polygons](https://en.wikipedia.org/wiki/Monotone_polygon)

The y-monotone approach is well explained in the book [Computational Geometry, Algorithms and Applications](http://www.cs.uu.nl/geobook/). This book was the main source of knowledge when implementing the first versions of the tessellator. There are also some good explanations scattered around the web. A popular tessellator implementation also based on monotone polygons is the GLU tessellator which [libtess2](https://github.com/memononen/libtess2) is based on (mentioned here for reference). Lyon's tessellator, while exploiting the same mathematical properties, chose a very different approach when it comes to the actual implementation of the logic and data structures. GLU's tessellator makes heavy use of half-edge meshes with several passes to solve different geometrical problems (one pass to separate the initial shape into non-self-intersecting shapes, another pass to separate these shapes into y-monotone shapes, another pass to tessellate these shapes, etc.), while lyon's tessellator does not store the connectivity information in a data-structure and computes everything in a single pass.

## Sweep line

Obviously, we can't simply take any three vertices and connect them to make a triangle until all vertices have been processed. A triangle's edge formed by connecting two vertices must not intersect other edges (and, of course, must not be outside the polygon), so we need to make sure somehow that these intersections don't occur. Testing each new edge for intersections against all other edges in the polygon would be much too expensive. We need to process vertices and edges in a way that provides us with interesting invariants so that we don't need to go through the entire geometry at every step.

The tessellation is a [[Sweep-line algorithm]] traversing the shape from top to bottom (by increasing y coordinate).

The sweep line can be seen as an imaginary horizontal line that scans the polygon from top to bottom stopping at each vertex (or event). In terms of code, the sweep line is represented as a sequence of pairs of edges that intersect the imaginary line. At each event the sweep line is updated to account for the edges that don't intersect it anymore and the edges that begin intersecting it. the edges on the sweep line are sorted by increasing x coordinate (from left to right).

![Animation illustrating a sweep-line scan](https://raw.githubusercontent.com/nical/lyon/master/assets/wiki/tessellator_sweep.gif)

The animation above illustrates a sweep line algorithm traversing a simple polygon vertex after vertex. Edges that intersect the sweep line are highlighted in orange and edges that are entering the sweep line are highlighted in blue.

Using a sweep-line strategy helps with reducing the complexity of the problem at hand. As we traverse the shape, we have a local understanding of its geometry (at the level of the sweep line and above) which gives us enough guarantees to be able to generate triangles that don't intersect with other edges without having to test against every edge in the shape.

A _Span_ is the area between two edges of the sweep-line representing the intersection between a piece of the inside of the and the sweep line.


# Ear clipping

# Constrained Delaunay triangulation