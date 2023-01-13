[Doc](https://learn.microsoft.com/en-us/windows/win32/direct2d/direct2d-portal)

## older versions

* Scanline conversation with trapezoids for edge AA. See [wpf-gpu-raster](https://github.com/jrmuizel/wpf-gpu-raster/blob/main/src/hwrasterizer.cpp).

## More recent versions

* More recent versions use tessellation with Target Independent Rasterization (TIR) which gives a coverage value for an edge which get summed into mask texture. It looks like the mask texture is atlased to allow batching multiple paths.
