---
Authors: Raph Levien
---

#tiling #compute #webgpu 

Link: https://github.com/linebender/vello/

Raph Levien's notes: https://docs.google.com/document/d/1LILagXyJgYtlm6y83x1Mc2VoNfOcvW_ZiCldZbs4yO8/
Blog posts: https://github.com/linebender/vello/blob/main/doc/blogs.md

Piet-metal's successor, previously named piet-gpu. A similar tile-based architecture, with a very similar rendering shader at the end of the pipeline, however the tiling process is much more sophisticated.

https://raphlinus.github.io/rust/graphics/gpu/2020/06/12/sort-middle.html

 - Inspired by [[High-performance software rasterization on GPUs.pdf]] ([Link](https://research.nvidia.com/publication/high-performance-software-rasterization-gpus))
 - Multi-stage tiling which first bins into large 256x256 pixels bins, and later in small 16x16 tiles.
 - Part of the reason for intermediate bins is to be able to use shared memory which has a limited size.
 - Binning in parallel breaks ordering of the commands, so the next stage ("coarse raster") has to sort them back. It then counts edges to allocate spaces for tile drawing commands before writing them out.

Main challenges:
 - Dynamically allocating memory to make space for drawing commands on the GPU.
 - Complex compute kernels, subject to the whims of poor drivers.

In my humble opinion this is the most promising approach to rendering large-scale vector graphics on *modern* GPUs. Emphasis on "modern" because it requires advanced compute shader features and it's unclear to me how well it would work on the zoo of buggy hardware and drivers out there. But on the high-end it is quite impressive. There is work underway towards a software fallback using the same shader code but compiled ahead of time to be run on the CPU with SIMD.

## Piet-metal

Source code: https://github.com/linebender/piet-metal Most interesting bits are in https://github.com/linebender/piet-metal/blob/master/TestApp/PietRender.metal (actually quite easy to read).

[Raph Levien](https://raphlinus.github.io/)'s first GPU path rendering experiment.

Architecture explained here https://raphlinus.github.io/rust/graphics/gpu/2019/05/08/modern-2d.html and here https://raphlinus.github.io/rust/graphics/gpu/2020/06/01/piet-gpu-progress.html

 - Entirely in compute shaders
 - Tile-based architecture (like pathfinder) with 16x16 pixels tiles.
  - The scene is packed into a big GPU-visible data structure with path commands.
  - A compute shader first assigns edges to screen-space tiles in a kind of brute-force way by having all wavefronts traverse the whole scene. Wavefronts are organized in blocks of 16x1 tiles so that the edges above and below the row can be quickly discarded in parallel by each thread of the wavefront, using wavefront instructions (ballot) before proceeding with the second part of the shader that assigns remaining edges to tiles horizontally.
  - The result of the tiling shader is for each 16x16 tile is a list of rendering commands that affect the tile:
    - Begin/end path + id of the path's pattern
    - edge (line segment)
    - a "backdrop" command indicating that the top-left corner of the tile is inside a path
  - Another compute shader, the rendering shader, reads these commands and render them into the framebuffer. 1 thread per pixel, 1 wavefront per tile.
    - What's interesting about this approach is that blending happens entirely in registers. Only the final pixel is written into the framebuffer. That's a very important performance win.

Drawbacks:

 - Dynamic allocation inside a shader is hard (don't know ahead of time how much space to reserve for tile commands).
 - The whole rendering model is implemented in the rendering shader. For example solid colors, gradients, images, blend modes, etc. are all done inside a switch statement when looping over the commands. Extending the shader with new features is always a risk of making the shader slower (more register hungry, etc.). However this could be mitigated by moving some functionalities into a pass before rendering which bakes any kind of pattern into an image so that the rendering shader treats it as an image.
 - (piet-metal only, does not apply to piet-gpu:) The tiling code is very simple but may not scale well for some scenes. Issues explained towards the end of this post: https://raphlinus.github.io/rust/graphics/gpu/2020/06/12/sort-middle.html
