---
aliases: RAVG
---

#tiling 

Link: https://hhoppe.com/proj/ravg/

Authors: Diego Nehab, Hugues Hoppe 

RAVG for short. 

Probably one of the most fundational papers in tile based gpu path rendering as it is well written, detailed and one of the firsts if not the first to go into a lot of details about this approach. 

Unlike some tile based approaches that bin tiles in screen space, this one bins in object local space resulting in a sort of acceleration structure for making sample queries in the tiled representation (in a random access fashion). This means that the result of the binning stage can be reused over multiple frames, however padding around tiles has to be included in order to sample near the tile boundary.

Another contribution is the quadratic bézier distance approximation which is fast to compute and sufficiently precise very close to the curve (good for anti-aliasing, not so much for general distance computation).


# Steps 

## CPU binning 
 
- Cubic bézier curves are approximated with quadratic béziers
- Bin the resulting edges into tiles (TODO: looks DDA-ish)

## GPU queries

For quadratic curves a horizontal ray intersection test is performed expressing the quadratic polynomial in the form `Qy(t) - Py = 0` to perform the in/out test and a fast distance approximation for the aa coverage. 

## Sampling considerations 

Simply projecting the sample in local space and evaluating the coverage there leads to blurriness under stretching or perspective transforms.

To mitigate that, the paper proposes prefiltering and supersampling. 

# Quadratic bézier distance approximation 

```C
inline float det(float2 a, float2 b) { return a.x*b.y-b.x*a.y; }
// Find vector 𝑣𝑖 given pixel 𝑝=(0,0) and Bézier points 𝑏0,𝑏1,𝑏2.
float2 get_distance_vector(float2 b0, float2 b1, float2 b2) {
float a=det(b0,b2), b=2*det(b1,b0), d=2*det(b2,b1); // 𝛼,𝛽,𝛿(𝑝)
float f=b*d-a*a; // 𝑓(𝑝)
float d21=b2-b1, d10=b1-b0, d20=b2-b0;
float2 gf=2*(b*d21+d*d10+a*d20);
gf=float2(gf.y,-gf.x); // ∇𝑓(𝑝)
float2 pp=-f*gf/dot(gf,gf); // 𝑝′
float2 d0p=b0-pp; // 𝑝′ to origin
float ap=det(d0p,d20), bp=2*det(d10,d0p); // 𝛼,𝛽(𝑝′)

// (note that 2*ap+bp+dp=2*a+b+d=4*area(b0,b1,b2))
float t=clamp((ap+bp)/(2*a+b+d), 0,1); // 𝑡̅
return lerp(lerp(b0,b1,t),lerp(b1,b2,t),t); // 𝑣𝑖 = 𝑏(𝑡̅)
```
