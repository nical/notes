Paper: TODO

This paper is very vague. The algorithm is described at a high level and there is no mention of whether binning is done on the gpu and if so what data structure for is used for storing the binned edges. 

The general idea is in the same vein as [[Pathfinder]]. Edges are binned into a regular tile grid, winding numbers are propagated and a shader tenders each tile. 

# Steps
## Binning step 

- Modified [[Digital differential analyzer]] to determine tiles that overlap the edge
- I suppose the edges are stored in a linked list but the paper does not say.


## Backdrop propagation 

The paper calls it spreadiong winding numbers. Basically a prefix sum

## Rendering 

A shader rasterizes the edges for the given tile like in piet or pathfinder compute