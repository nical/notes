There is a growing trend of splitting rendering into small fixed-size tiles internally in modern game engines. For example see idTech 6's shadow atlas, texture-space shading of the particles, and more generally the sparse virtual texture, lots of other engines are also using similar tricks. FastUIDraw also uses a tiled atlas for its image cache.

There are some nice advantages of organizing image data this way:

-   Managing the atlas memory becomes very easy and space-efficient, regardless of the sizes of the elements in the atlas.
-   culling fixed size tiles is easier than variable sized elements.
-   If there is a lot of per-tile work to do on the CPU, it is often possible to do it in parallel
-   It becomes easy to only allocate atlas memory for visible tiles of a resource
-   managing invalidation and partial updates in the atlas is simpler. idTech6's shadow don't recompute tiles that have not changed since the previous frame.

One downside is that most implementation have to use an indirection texture adding an extra texture lookup. It's also important to be careful about sampling artifacts at tile boundary (a lot of implementations seem to add 1px padding around each tile).

For 2d rendering, this decoupling between an object's total size and the portion that is actually rendered is particularly interesting. If a big object is several times larger than the viewport, all of the memory and work going into processing content that is off the viewport is wasted. Working at the tile level makes culling easier and removes the object size from the equation.