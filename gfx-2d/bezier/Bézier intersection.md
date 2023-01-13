
# Cubic vs cubic

Fat-line clipping.
A "fat line" is the two lines, one on each side of the curve's base line, between the curve is entirely contained. Ideally a fat line is as tight as possible.

The idea is to clip each bezier curve against the fat-line of the other until the fat lines convere to simple lines and retrieve their intersections.

# Quadratic vs line

Analytical 
Skia code: https://github.com/google/skia/blob/main/src/pathops/SkDQuadLineIntersection.cpp

