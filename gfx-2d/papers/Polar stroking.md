
#paper 

Authors: Mark Kilgard

# Overview

Provides some maths for finding the flattening edge density of curves in the context of tessellating skeletal strokes.

It allows placing an upper bound on the curvature per segment (as opposed to just using the distance error as the flattening criterion).

# Intuition

![[polar-stroking-quad.png]]
(Figure from the paper)

when plotting the hodograph (the derivative of the curve), the closer we are to zero the coser we are to a cusp, and more generally bézier curves tend to have less velocity where their curvature is important.
using polar coordinates for positions on the hodograph we have a relation between the variation in angle of said polar coordinate and the curvature of the curve at the corresponding t.

