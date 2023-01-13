
* [[Stencil and cover]]
* [[Vertex-AA]]. Much easier with stencil-and-cover than with tessellation because they can just extrude the aa "bands" and inset the stenciled geomerty without worrying about self-overlap and folding.

- `nvgFill` does some geometry manipulation, calls into TODO which pushes some uniform state and memcpy the vertices into a buffer
- `glnvg__renderFlush` (probably called at the end) actually submits draw calls for each path by calling into `glnv__fill`, `glnv__stroke` in a loop.
 - The vertex-aa is called "fringe" in the code, it goes into its own draw calls
- If the stroke width is inferior to 1px, stroke coverage is simulated using transparency
- No draw call batching, vertex and uniform data is grouped into large buffers 
- There is a fast path for convex fills 
- Strokes are either teaaellated or drawn with the stencil buffer.