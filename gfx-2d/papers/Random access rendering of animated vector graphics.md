#paper #tiling 

Authors: Ivan Leben

Followup to [[Random access vector graphics]] (by different people) moving the binning phase to the gpu while keeping a relatively small gpu feature set (no compute). 

Present a few attempts at replacing RAVG's CPU binning algorithm with alternatives that run on the GPU.

# Pivot-point localized image representation 

The general idea is that the winding number of any point in a tile can be known if we know the winding number of a point (the pivot) in the tile and ray cast from there.
They use the center of the tile as the pivot point computed during binning.
The main advantage is to not need auxiliary edges. 

## Binning 

A quad per segment is rendered into a framebuffer with a pixel per tile. The fragment shader writes into a buffer instead of writing into a render target.
