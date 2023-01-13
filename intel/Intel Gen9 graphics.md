Skylake
2016

[[Gen9 performance guide.pdf]]

# Capabilities

- DX12
- Vulkan 1.3 (Windows) 1.2 (Linux)
- GL 4.6
- GLES 3.2
- OpenCL 3.0
- [AST](https://fr.m.wikipedia.org/w/index.php?title=S3TC&action=edit&redlink=1 ) C texture compression 

# Architecture 

- Each thread in the Intel® Processor Graphics Gen9 architecture has 128 registers and each register is 8 x 32 bits. For shaders other than fragment shaders, keep in mind that each register represents a single channel of vec4 variable for 8 data streams.
- Fragment shaders can be launched by the driver in SIMD8, SIMD16 or SIMD32 modes. Depending on the mode, a single channel of vec4 variable can occupy a single register, or two or four registers for 8, 16, or 32 pixels.
- Registers are used for the delivery of initial hardware-dependent payload as well as uniforms, which can consume up to 64 registers. A bigger number of variables and uniforms in shaders will increase register pressure, which can lead to saving/restoring registers from memory. This, in turn, can have a negative impact on the performance of your shaders. 
- Limit the number of uniforms in the default block to minimize register pressure; however, the default uniform block is still the best method of constant data delivery for smaller and more frequently changed constants.
- Don’t use compute shader group sizes larger than 256 (and recall that largergroups will require a wider SIMD). They result in higher register pressure and may lead to EU underutilization.
- Textures with power-of-two dimensions will have better performance in general.
- Shader Storage Buffer Objects present a universal mechanism for providing input/output both to and from shaders. Since they are flexible, they can also be used to fetch vertex data based on gl_VertexId. Use vertex arrays where possible, as they usually offer better performance.



# Memory

# Clears

Use RGBA `[0,0,0,1]` clears for best performance, but use any 0/1 clear value when possible. Use `VK_ATTACHMENT_LOAD_OP`+`CLEAR` to enable this rather than `vkCmdClearColorImage`.

# Compute

Frequently switching between 3D and Compute pipelines might incur significant context-switching overhead. While it is defined by the API to allow for asynchronous dispatch of 3D and compute, it is not recommended to structure algorithms with the expectation of latency hiding by executing 3D and compute functions simultaneously. Instead, structure algorithms to allow for minimal latency/dependency between 3D and compute capabilities.

# Misc
- If compute kernels are very short, thread launch can be an overhead. Try forcing a higher SIMD, or introduce a loop so there is more work per thread.
- Data reads are done at a 64-byte granularity. Using an uchar data type can result in partial reads and writes of cache lines, which can affect performance. Use a wider data type like int or int4.

- For texturing, use texture arrays. They provide a much more efficient way of switching textures compared to reconfiguration of textureunits.
- By using instancing and differentiating, rendering based on gl_InstanceID can also provide a high-performance alternative to rendering similar objects, when compared to reconfiguring the pipeline for each objectindividually.
- Use the default uniform block rather than uniform buffer objects for small constant data that changes frequently.
- Limit functions that switch frame buffer objects and GLSL programs. They are the most expensive driver operations.
- Avoid redundant state changes.
- In particular, do not bind a state to default values between draw calls, as not all of these state changes can be optimized in the driver
- Minimize per-sample operations, and when shading in per-sample, maximize the number of cases where any kill pixel operation is used (for example, discard) to get the best surface compression.
-  Minimize the use of stencil or blend when MSAA is enabled.
- Avoid creating shaders where the number of temporaries (dcl_temps for D3D) is ≥ TODO

- Structure code to avoid unnecessary dependencies (especially dependencies on high-latency operations like sample).
- Avoid flow control decisions on high-latency operations. Structure your code to hide the latency of the operation that drives controlflow.
- Avoid flow control decisions using non-uniform variables, including loops. Try to ensure uniform execution among the shader threads.
- Avoid querying resource information at runtime (for example, the High Level Shading Language (HLSL) GetDimensions call) to make decisions on controlflow, or unnecessarily incorporating resource information into algorithms; for example, disabling a feature by binding an arbitrarily sized (1 x 1) surface. 
- Implement fast paths in shaders to return early in algorithms where the output of the algorithm can be predetermined or computed at a lower cost of the full algorithm.
- Use discard (or other kill pixel operations) where output will not contribute to the final color in the render target. Blending can be skipped where the output of the algorithm is an alpha of 0, or adding inputs to shaders that are 0s that negateoutput.
- UBO accesses have two modes, direct (offset known at compile time) and indirect (offset computed at runtime). Avoid defining algorithms that rely on indirect accesses, especially with control flow or tight loops. Make use of direct accesses for high-latency operations like control flow andsampling.