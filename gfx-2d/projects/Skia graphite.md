
Link: https://github.com/google/skia/tree/main/src/gpu/graphite

 - [[Multi-sampling]] (MSAA)
 - [[Stencil and cover]]
 - [[2D occulsion culling]] (Depth buffer)


Multiple rendering techniques implemented (for example “SDF glyph” or “tessellated stroke”). Each rendering technique has a singleton Renderer which holds a series of RenderStep. RenderStep is where the technique specific code lives. Render steps can be reordered to improve batching (very webrender-like). See graphite/Renderer.h

  

There is a render task graph with automatically atlased render tasks like WebRender.

See Task and the classes that inherit from it. RenderPassTask is probably the most interesting one, also note that texture uploads are tasks.

  

Interesting comment taken directly from DrawList.h:

```
A DrawList represents a collection of drawing commands (and related clip/shading state) in a form that closely mirrors what can be rendered efficiently and directly by the GPU backend (while balancing how much pre-processing to do for draws that might get eliminated later due to occlusion culling).

A draw command combines:
 - a shape
 - a transform
 - a primitive clip (not affected by the transform)
 - optional shading description (shader, color filter, blend mode, etc)
 - a draw ordering (compressed painters index, stencil set, and write/test depth)

Commands are accumulated in an arbitrary order and then sorted by increasing sort z when the list is prepared into an actual command buffer. The result of a draw command is the rasterization of the transformed shape, restricted by its primitive clip (e.g. a scissor rect) and a depth test of "GREATER" vs. its write/test z. (A test of GREATER, as opposed to GEQUAL, avoids double hits for draws that may have overlapping geometry, e.g. stroking.) If the command has a shading description, the color buffer will be modified; if not, it will be a depth-only draw. In addition to sorting the collected commands, the command list can be optimized during preparation. Commands that are fully occluded by later operations can be skipped entirely without affecting the final results. Adjacent commands (post sort) that would use equivalent GPU pipelines are merged to produce fewer (but larger) operations on the GPU. Other than flush-time optimizations (sort, cull, and merge), the command list does what you tell it to. Draw-specific simplification, style application, and advanced clipping should be handled at a higher layer.
```

  
  

ClipStack::applyClipToDraw do intersection checks to see if the clip has any effect at all on the clipped primitive

They use a simplified skyline atlas allocator (for glyphs at least), It does not seem to have a way to remove items. (See Rectanizer.h). TODO: how do they incrementally maintain the cache?

## Interesting stacks (TODO):

```
Device::drawPath
	Device::drawGeometry (all high level primitives go through here)
		Device::chooseRenderer
		    DrawContext::recordDraw
		        DrawList::recordDraw
        ...

Device::flushPendingWorkToRecorder
	DrawContext::snapRenderPass (snap really just means flush)
	    DrawContext::snapDrawPass
	        DrawPass::Make (sorting happens here)
```

  

DrawPass::Make sorts the draws, that sorting looks like the main element of batching.

The sort key contains information about the render step and bound resources. Then the commands are recorded in a DrawWriter

Device::chooseRenderer is interesting to look at, it’s where we decide what rendering technique to use for a given high level primitive.

## The renderers

Renderers are sequences of render steps.
-   Analytic rounded rects (no msaa)
-   Bitmap text (used for most glyphs) (no msaa)
-   SDF text (used rarely) (no msaa)
-   Stencil tessellated curves and tris
-   Stencil fan (requires msaa)
-   Stencil curve (requires msaa)
-   Cover
-   Stencil tessellated wedges
-   Stencil wedge (requires msaa)
-   cover
-   Convex tessellated wedges (requires msaa)
-   Tessellated strokes (requires msaa)
-   Vertices (pre-triangulated geometry) (does not require msaa)

TessellatedCurvesRenderStep writes a flattened version of the path directly on the GPU (no sweep line triangulation step)

Prefer wedges if number of verbs is low and draw area < 256*256

A good way to check some aspects of each render step (for example whether they rely on msaa or if they perform shading) is to look at the flags specified in the ctor of each render step.

## Batching

Batching happens somewhat implicitly. It is done by having consecutive calls to RenderStep::writeVertices just push compatible instances. So the ordering is really what drives the effectiveness of batching. See DrawOrder.h. When primitives are inserted, they explicitly state dependencies via a DrawOrder object (for example whether they depend on the front-to-back “painter’s” order, on stencil, etc.) which represent the reordering constraints. The interesting parts affecting batching is how this order object is manipulated in Device::drawGeometry (which happens very early as opposed to webrender which puts the batching pass at the end of frame building).

The ordering constraints affect each draw’s sort key and the real command ordering is derived from that key via a standard sort in DrawPass::Make. The comment on top of DrawPass::SortKey in DrawPass.cpp has interesting details. 

There is a BSP tree used to allow draws depending on stencil to be reordered as long as they don’t overlap.

# Thoughts

## WebRender-like

The approach of looking at the frame globally and scheduling work via a render task graph looks very WR-like. In any case it’s much much closer to webrender than the current way other skia backends simply process drawing commands one by one.

Some key differences include:

-   The scope: we do layerization and invalidation in WebRender, while skia just needs to render the whole set of drawing commands specified for a SkCanvas, and layerization/partial rendering is implemented on top of it.
-   WebRender processes an entire window at a time, while graphite’s granularity is the SkCanvas which would map to a Layer/Picture cache slice in our case, although some caches are shared between multiple SkCanvases (glyphs, etc.).
-   As a result, no scene building, no visibility pass, no plane splitting.
-   They have well defined abstraction layers for adding new ways to render things, see the “extensibility” section below.
-   Obviously filling and stroking arbitrary primitives is one of the core primitives while we are still very much focused on rendering (rounded) rectangles in WR.

## Depth buffer

The depth buffer is used (and required) probably for perf but also for correctness as it’s how they ensure self-overlapping strokes with transparency can render without double-blending.

## Msaa

Most of the anti-aliasing is done via msaa (exceptions are text of course, analytical rounded rects and the vertices render step. Not sure whether the latter gets used without the user opting in to passing pre-triangulated geometry as input). A surprising choice given how slow msaa is on many or all intel iGPUs.

Another interesting aspect of their use of MSAA is what they call “DMSAA” for dynamic MSAA. The idea is to render only some sequences of commands using MSAA. At the end of an MSAA pass of draws the temporary MSAA buffer is resolved back into the single sampled surface.

“some memory/performance wins on platforms/apis that support memoryless textures (gl render to texture, vulkan lazy allocated memory, etc.) [..] we can simply discard all the memory after render passes and thus the gpu never needs to actually allocate real memory.

## No fast simple non-aa rectangle primitive?

I don’t see a simple non-aa rectangle code path which is the bulk of what WebRender does (unless the analytical rounded rect render step counts as one).

## Stencil and cover

Convex fills require stencil and cover. Non-overlapping stenciled shapes are batched but not overlapping ones.

## Culling

Occlusion culling is done by the BoundsManager (See BoundsManager.cpp and how it is used in Device.cpp).

There are different algorithms implemented:

-   NaiveBoundsManager (no culling)
-   BruteForceBoundsManager (tracks every draw, exact culling via brute-force search, sounds like WebRender’s batching code)
-   GridBoundsManager (uses a uniform spatial grid)
-   HybridBoundsManager (used by default, first bruteforce up to a limit then switches to grid)

## local/global optimizations

At first glance it looks like skia went from a pretty immediate/local API where each command is executed individually and if need be a series sequence of primitives on a given target is interrupted to paint something in an intermediate target, to something a bit more “global” à la WebRender where a render task graph is built for a larger scope than a single primitive and intermediate targets are pre-rendered.

I think that it’s a better model in general. There are pathological cases that WR handles poorly when an unreasonably large amount of pixels must be pre-rendered and we run out of memory whereas skia used to not need to allocate a gigantic amount of memory. My understanding from the big comment in DrawAtlas.h is that graphite will accumulate atlased intermediate results in one or more atlases up to a certain limit after which adding a new atlas item will fail, giving the device a chance to end the current draw, flush the render pass task and start a new one. In other words it looks like it has the best of both worlds: avoid flipping back and forth between intermediate and final targets up to a certain limit to also avoid unbounded memory for intermediate targets.

## Extensibility

I like how extensible their abstraction is in comparison with WR. They have a pretty black box abstraction for “render something into a target” which does not appear to be tied to a particular vertex format, while still being able to batch consecutive commands of the same type.

WebRender on the other hand requires adding code in a dozen different places when adding a new type of pattern. The vertex format when rendering in a picture can only be indexed quads with an ivec4 per instance otherwise it does not integrate with our batching infrastructure, and while other render tasks are more free-form we have to implement the feature twice if we want to be able to render something both in render tasks and in pictures.

See graphite/Renderer.h for the renderer abstraction. The interesting bits are implemented by deriving RenderStep

## Multi-threading

Saw a mention on the skia mailing list that one of the goals of graphite is to be able to dispatch drawing commands (the GPU API ones) on multiple threads.