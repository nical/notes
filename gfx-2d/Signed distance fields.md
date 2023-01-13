Original paper: https://steamcdn-a.akamaihd.net/apps/valve/2007/SIGGRAPH2007_AlphaTestedMagnification.pdf

See also:
- Iq:
	- [2D signed distance functions (iq)](https://iquilezles.org/articles/distfunctions2d/)
	- [other post](https://iquilezles.org/articles/distgradfunctions2d/)
	- [L-norms](https://iquilezles.org/articles/distfunctions2dlinf/)
- Shadertoy:
	- quadratic beziers SDF [shadertoy.com/view/dls3Wr](https://www.shadertoy.com/view/dls3Wr "https://www.shadertoy.com/view/dls3Wr")
	- polygon SDF https://www.shadertoy.com/view/wdBXRW
- [Practical analytic 2D signed distance field generation](https://dl.acm.org/doi/10.1145/2897839.2927417)
- http://www.essentialmath.com/GDC2015/VanVerth_Jim_DrawingAntialiasedEllipse.pdf
- How Skia renders SDF textures (tutorial): [Part 1](https://www.essentialmath.com/blog/?p=111), [Part 2](https://www.essentialmath.com/blog/?p=128), [Part 3](https://www.essentialmath.com/blog/?p=151).
- [Multi-channel Signed Distance Fields](https://github.com/Chlumsky/msdfgen)


-   [Adaptively Sampled Distance Fields](https://graphics.stanford.edu/courses/cs468-03-fall/Papers/frisken00adaptively.pdf) (proprietary and patented)
    -   Uses an octtree or quadtree to represent a sampled SDF: higher resolution and more tree levels where the rate of change is higher (eg near corners). Saffron adds patented extensions: a biquadratic approximation of the sampled distance field and an explicit representation of the distance field around sharp corners. This combination reduces the lookup and storage cost by using multiple cell types.
    -   [A New Framework for Representing, Rendering, Editing, and Animating Type](http://ronaldperry.org/SaffronTechDocs/Saffron_Paper_SIGGRAPH_Submission.pdf) 2002, from the [Saffron project](http://ronaldperry.org/SaffronTechDocs/).
    -   [Nitro (Saffron successor) on GPU outperforms FreeType on CPU with much simpler code](http://ronaldperry.org/SaffronTechDocs/Nitro_Versus_FreeType.pdf) 2014.
    -   Note: Other hierarchical sampled SDF structures with better GPU support: VDB, mipmap

	- Generating SDFs on the GPU: https://astiopin.github.io/2019/01/06/sdf-on-gpu.html