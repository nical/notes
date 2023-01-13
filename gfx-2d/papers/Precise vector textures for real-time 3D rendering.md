#paper #tiling 

Some similarities with [[Random access vector graphics]]

Tiling scheme used as an acceleration structure for random access sampling

Curves are split into monotonic parts

Proposes a quadratic bézier distance computation based on binary search (can be used on any kind of parametric curve). I suspect that it relies on the curves being monotonic. It can give incorrect results when the distance is larger than the curvature radius. 

The in/out test looks different from RAVG. It looks like winding numbers are already taken into account when encoding the monotonic "features" that are binned into the tiles such that we only need to know which side we are of the feature to perform the in/out test instead of accumulating winding numbers within the tile. That would suggest that self-intersections are removed beforehand.

They measure worse performance than RAVG overall. 