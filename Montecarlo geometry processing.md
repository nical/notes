Paper: TODO

A random walk algorithm to determine properties at any point in space from boundary information. For example it can work as an in-out test.

It works on degenerate (non-closed) shapes.

The random walk is similar to sphere-tracing/ray-marching:
 - determine the distance `d` to the closest boundary point
 - if `d < epsilon` return the information at the current location
 - else move to a location `d` away from the current location in a random direction
 - start over.

A number of random walks (samples) can be done, then averaged. The more samples, the less noise.
Other denoising techniques can be used to improve the output without doing a very large number of samples per location.

Very explanation at https://blog.demofox.org (TODO: exact link)
