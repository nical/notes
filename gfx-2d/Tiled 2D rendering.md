#tiling 

A lot of recent attempts at doing vector graphics on the GPU revolve around breaking large and complex scenes into a regular grid of small (for example 16x16 pixels) tiles.

Very important paper to read (maybe the first to explore this tiling idea): [[Random access vector graphics]] or in short "the RAVG paper".

Interesting properties of working with tiles:
 - Break complex scenes into small manageable parts. It would be unreasonable to have a shader loop over all edges of a scene for each pixel, however it's totally reasonable to have it loop over all of the edges intersecting the 16x16 tile that the pixel is in.
 - Regular tile grids are easy to work with and wrap our heads around. Especially on GPU, the "regular" aspect is key to efficiently doing work in parallel. More involved data structures may offer better algorithmic complexity at the expense of worse parallelism, memory access patterns, register usage, etc.
 - Easy to notice that a tile is entirely covered by an opaque path and discard all of the content that is underneath.
 - Mos of these approaches have a "tiling phase" and a "rendering phase", with a data format in between. The tiling phase usually the most challenging. It's rather easy with this architecture to have different implementations for the tiling phase (potentially falling back to CPU) and pick depending on hardware/driver features and bugs.

I couldn't find source code for the original RAVG paper, however there is a [thesis](https://docs.google.com/file/d/0B-YD8rWEqGgaZE5BY1NKaDBiNjA/edit?resourcekey=0-CfpA9oHUHXHMp-hzn5Igpg) by Ivan Lebel (link on [his blog](https://ivanleben.blogspot.com/2010/12/animated-random-access-vector-graphics.html)) on followup RAVG work with source code at [https://github.com/ileben/RAVG](https://github.com/ileben/RAVG).

# Publications

 - [[Random access vector graphics]]
 - [[Random access rendering of animated vector graphics]]
 - [[Precise vector textures for real-time 3D rendering]]

# Projects

 - [[Vello]]
 - [[Forma]]
 - [[Pathfinder]]
 - [[Spinel]]
 - [[Ochre]]

