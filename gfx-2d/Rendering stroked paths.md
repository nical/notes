
# Using meshes

Simple/common cases are easy to handle but some configurations are challenging because it is difficult to produce geometry without self-overlaps.

Cases where the approach fits well include:
 - Fully opaque strokes (no need to deal with self-overlap).
 - Using the depth buffer to prevent self-overlap (with [[Multi-sampling]]).
	 - Done in [[Skia graphite]].



See also: [[Polar stroking.pdf]].

# Stroke-to-fill conversion

A more general approach that deals with every cases, can be the only solution or a fallback.

- curve offsetting
- walk the contour of the offset path
	- no need to deal with self-intersections, non-zero fill rule takes care of that
	- can still avoid some of the obvious cases of self-intersections as an optimization

