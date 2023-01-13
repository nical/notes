#tiling #compute #webgpu

Link: github.com/google/forma

See also: [[Spinel]]

- WebGPU
- Compute shaders
- GPU sort
- GPU raster
- Uses [etagere](crates.io/crates/etagere) for texture atlases
- CPU fallback
- Uses Raph Levien's [[Bézier flattening]] algorithm
- Flattened segments are cached and reused across translations and rotations

TODO: 16x4 tiles ?

4 stages:
- Flattening (curves -> line segments)
- Rasterization (line segments -> psegments)
- Sorting (psegments -> sorted psegments)
- Painting (sorted psegments -> painted tiles)

Works with a full-scene approach where a representation is built on the GPU and the screen is filled in a single dispatch that understands the entire drawing model (like [[Vello]]). The binning strategy consists in first generating a segment representation (psegment, presumably for "pixel-segment") which is then sorted. The sort key of the segment contains the tile index which means that after sort, a tile's content is encoded in a continuous range of the psegment buffer. This is in contrast with piet which will more explicitly bin edges into tiles (here the binning is implicitly provided by the sort).


Tiles determine the starting offset after sort using a binary search (see findStartOfTileRow)

```
Idea: instead of having each tile do a binary search over the sorted buffer, could add a pass that in parallel scans blocks (with 1 element of overlap at the start) of the sorted buffer and assigns to an indirection table the beginning of a tile's data when the tile index changes in the stream.
It adds a sparse intermediate data structure but probably reduces the number of reads by a lot with large scenes since the is no O(log(n_segments))) search to perform per tile
```

The compute shader appears to scan entire rows of tiles, probably why I'm not seeing any logic for backdrop propagation. 

## Rasterization

The generation of the psegment is called the rasterization phase. It takes care of some of the per-segment parts of the computations (so that it isn't done per-pixel in a later stage).

psegment encodes into a 64 bits integer: (TODO: check, this is copied from cassia notes)
  - tile x, y
  - local x, y (within tile) 
  - layer index
  - area
  - coverage

## Sort

TODO

## Painting

The phase that writes into the render target is called "painting".

compute shader goes:

```
main() {
    accumulate cover from psegments with negative x

	while tile.x < tile_row_len {
	    paint_tile(tile)
	    tile.x += 1;
	}
}

paint_tile() {
    loop {
	    read psegment
	    if current layer != segment layer {
		    blend_layer()
	    }

		pop_queue_until(layer) // also calls blend_layer() for each poped layer

		if psegment.local_pos == local_pos {
			double_area = // some computation based on segment
			cover += // some computation based on segment
		}
    }
}

blend_layer() {
    switch type {
        color:
            // ...
        gradient:
            // ...
        etc.
    }
}
```

- double area: TODO
- cover:




# Cassia

Notes below applied to cassia which appears to be have become forma, probably applies to forma as well

There's a rust/webgpu port of some of the ideas behind spinel here: https://github.com/Kangz/cassia It's a bit easier to digest but a large part of it is running on the CPU instead of the GPU.

The following are observations about cassia, some of it may apply to spinel:
 - Instead of rasetrizing line segments at the end of the pipeline, edges are broken up into "psegments" which I assume stands for pixel-segments. These encode the following into a single 64 bit integer:
  - tile x, y
  - local x, y (within tile, only 3 bits per axis, tiles are 8x8 pixels) 
  - layer index
  - area
  - coverage
 - There's a sorting pass using the value itself (taking advantage of the tile y, x being at the top the psegment).
 - See [surpass/src/rasterizer/mod.rs](https://github.com/Kangz/cassia/blob/main/surpass/src/rasterizer/mod.rs) and [surpass/src/segment.rs](https://github.com/Kangz/cassia/blob/main/surpass/src/segment.rs) for the psegment building part (CPU-side).
 - See [cassia/src/TileWorkgroupRasterizer.cpp](https://github.com/Kangz/cassia/blob/main/cassia/src/TileWorkgroupRasterizer.cpp) for the rasterization (GPU-side).

