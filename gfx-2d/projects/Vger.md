
Source code: https://github.com/audulus/vger-rs and https://github.com/audulus/vger

# The simple renderer
  * On the CPU, paths are decomposed into quadratic bézier curves and binned into horizontal bands via a simple sweep,
  * a fragment shader rasterizes the bands
  * edges are not flattened, they are rendered as quadratic bézier curves using the ravg distance approximation for aa and something inpsired from loop-blinn for the inside/outstde test.


# The experimental tiled renderer
- probably abandonned
* edges (and other types of primitives) are assigned to tiles in a shader,
* in another pass, pixels are rendered by going over the per-tile commmands (like piet's k4).
