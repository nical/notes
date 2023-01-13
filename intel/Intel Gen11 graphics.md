Icelake
2019

[[Gen11 performance guide.pdf]]

# Capabilities



# highlights 

Gen11 offers improved performance and efficiency over Gen9, and new features such as coarse pixel shading, tile-based rendering, and new display controller features. In addition, Gen11 offers the following improvements over previous generations: 
- Compute capabilities that deliver up to a teraflop of performance
- Significantly lower shared local memory (SLM) latency
- Larger L3 cache size
- Increased memory bandwidth
- Improved multisample anti-aliasing (MSAA) performance

# Tiling

Gen11 implements a tile-based rendering solution known as position only shading tile-based 
rendering (PTBR). The motivation of tile-based rendering is to reduce memory bandwidth by 
efficiently managing multiple render passes to data per tile. In order to support tile-based 
rendering, Gen11 adds a parallel geometry pipeline that acts as a tile binning engine. It is used 
ahead of the render pipeline for visibility binning pre-pass per tile. It loops over geometry per tile 
and consumes visibility stream for that tile. PTBR uses the L3 cache to keep per tile data on die, 
reducing external memory bandwidth. For more information, refer to the architecture guide or 
talk with an Intel application engineer to see if your workload will benefit.

- Only use trilist or tristrip topologies.
- For DirectX 12, use ID3D12GraphicsCommandList4::EndRenderPass with D3D12_RENDER_PASS_ENDING_ACCESS_TYPE_DISCARD.
- For Vulkan, Use VkRenderPass/VkSubpass with VK_ATTACHMENT_STORE_OP_DONT_CARE.
- Avoid tessellation, geometry, and compute shaders. Passes with tessellation and geometry shaders will not benefit from PTBR hardware improvements.
- Avoid intra-render pass read after write hazards.
- Separate the attributes required to compute position into separate vertex buffers.


# Perf recommandations 

## SSBOs and UAVs

- These resource types may cause inefficient partial writes over the Gen11 64-byte cache lines. Avoid these partial writes to get maximum bandwidth through the cache hierarchy. This can be done by ensuring that a single thread executing a given shader on a 4x2  group of pixels writes a contiguous 64 bytes on its own for output.
- Access to read-only data is much more efficient than read/write data. Use these kinds of resources with caution and when there are no better options.
- Do not set a resource to use a UAV bind flag if the resource will never be bound as a UAV. This programming behavior may disable resource compression.

## Anti-aliasing 

- Minimize the use of stencil or blend when MSAA is enabled.
- Avoid querying resource information from within a loop or branch where the result is 
immediately consumed or duplicated across loop iterations.
- Minimize per-sample operations. When shading per sample, maximize the number of 
cases where any kill pixel operation is used to get the best surface compression.
We do recommend using optimized compute shader post-processing anti-aliasing such as

## Clearing

Use the API provided functions for clear, copy, and update operations, and refrain from 
implementing your own versions. Drivers have been optimized and tuned to ensure that 
these operations work with the best possible performance.
- Enable hardware ‘fast clear’ values as defined per API: 
	- In DirectX 12, clear values are defined at resource creation as an argument with ID3D12Device::CreateCommittedResource.
	- For Vulkan, use VK_ATTACHMENT_LOAD_OP_CLEAR and avoid using vkCmdClearColorImage.
	- For other APIs, use (0,0,0,0) or (1,1,1,1).
	- Ensure horizontal alignment = 128b and vertical alignment = 64b.
- Copy depth and stencil surfaces only as needed instead of copying both unconditionally;  they are stored separately on Gen11.
- Batch blit and copy operations.

## Geometry
- Vertex fetch throughput is six attributes per clock (and max two vertices per clock). Ensure all bound attributes are used. When a draw is bottlenecked on geometry work, reducing the number of attributes per vertex can improve performance.
- Define input geometries as a structure of arrays for vertex buffers. Try to group position information vertex data in its own input slot to assist the tile binning engine for tile-based rendering. 
- The Gen11 vertex cache does not cache instanced attributes. For instanced calls, consider loading attributes explicitly in your vertex shader. 

## Shaders

- Structure the shader to avoid unnecessary dependencies, especially high latency operations such as sampling or memory fetches. 
- Avoid shader control flow based on results from sampling operations.
- Aim for uniform execution of shaders by avoiding flow control based on non-uniform variables.
- Implement early returns in shaders where the output of an algorithm can be predetermined or computed at a lower cost of the full algorithm.
- Use shader semantics to flatten, branch, loop, and unroll wisely. It is often better to explicitly specify the desired unrolling behavior, rather than let the shader compiler make those decisions. 
- Branching is preferable if there are enough instruction cycles saved that outweigh the cost of branching. 
- Extended math and sampling operations have a higher weight and may be worth branching (see Figure 6 for issue rate). Small branches of code may perform better when flattened.
- Unroll conservatively. In most cases, unrolling short loops helps performance; however, unrolling loops does increase the shader instruction count. Unrolling long loops with high iteration counts can impact shader residency in instruction caches, and therefore negatively impact performance. 
- Avoid extra sampler operations when it is possible that the sampler operation will later 
- be multiplied by zero. For example, when interpolating between two samples, if there is 
- a high probability of the interpolation being zero or one, a branch can be added to speed up the common case and only perform the load only when needed.
- Avoid querying resource information at runtime; for example, High-Level Shading Language (HLSL) GetDimensions call to make decisions on control flow, or unnecessarily incorporating resource information into algorithms.
- When passing attributes to the pixel shader, mark attributes that do not change per vertex within a primitive as constant.
- For shaders where depth test is disabled, use discard (or other kill operations) where output will not contribute to the final color in the render target. Blending can be skipped where the output of the algorithm has an alpha channel value of zero, or adding inputs into shaders that are zeros that negate output.

## Texture sampling 

- When sampling from a render target, avoid sampling across mip levels of the surface with instructions such as sample_l/sample_b.
- Use API defined and architecture supported compression formats (that is, BC1-BC7) on larger textures to improve memory bandwidth utilization and improve memory locality when performing sampling operations.
- Avoid dependent texture samples between sample instructions. For example, avoid making the UV coordinates of the next sample operation dependent upon the results of the previous sample operation. In this instance, the shader compiler may not be able to optimize or reorder the instructions, and it may result in a sampler bottleneck.
- Avoid redundant and duplicate sampler states within shader code, and use static/immutable samplers, if possible.
- Define appropriate resource types for sampling operation and filtering mode. Do not use volumetric surface as a 2D array.
- When fetching from an array surface, ensure that the index is uniform across all single instruction, multiple data (SIMD) lanes.
- Avoid defining constant data in textures that could be procedurally computed in the shader, such as gradients.
- Avoid anisotropic filtering on sRGB textures. 
- Sample_d provides gradient per pixels and throughput drops to one-fourth. Prefer sample_l unless anisotropic filtering is required.

