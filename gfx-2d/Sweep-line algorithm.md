https://en.wikipedia.org/wiki/Sweep_line_algorithm

# Problem 

A lot of geometry algorithm are, in their naive form `O(n^2)` oblver the number of edges.
Typically there is `n` for visiting each edge or vertex, and at each step, the need to compare some property of the edge against surrounding edges (for example check for intersections) which brings another factor of `n`.

Sweep lines exploit geometrical properties of the algorithm to bring the second factor of `n` to a lower value. 

# Above/below relation

See [[Float precision]] issues.

![Animation illustrating a sweep-line scan](https://raw.githubusercontent.com/nical/lyon/master/assets/wiki/tessellator_sweep.gif)

# Algorithms

![[Bentley Ottmann algorithm]]