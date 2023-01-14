For a some of applications, it is simpler or event necessary to approximate bézier curves with séquences of line segments. This opérations is called flattening. There are a number of ways to approach flattening bézier curves with different tradeoffs. Selecting the right method is important as it can have a large performance impact, in part because the of the cost of computing the approximation, and also because some methods will generate more edges than others, given the same approximation threshold.

Since we are in the realm of approximations, there are tradeoffs that can be made between the quality and runtime performance of the operation. Here we'll only consider the quality of a flattening approximation by evaluating the maximum distance between the approximated polyline and the original curve. This maximum distance will be referred to as the "tolerance threshold" or just "tolerance" parameter.
There exists flattening algorithms that also include maximum angle, constant parameter speed along the curve, or even incorporate curve offsets (for stroking), but the only tuning parameter I'm looking into here is the tolerance.

Some algorithms have two orthogonal concepts:
 - the core algorithm itsel (recursion, iterative process, etc.)
 - the flatness criterion, which in some case is interchangeable.



# Algorithms

## Recursive subdivision

- Very simple algorithm.
- Can work with different flattening criterions
- Not terrible but not great either at minimizing the number of edges. Both the performance and the number of edges depend greatly on the selected flatness criterion.

## Fixed step

A conceptually simple way to flatten the bézier curve is sample the curve at fixed intervals. In practice, finding the minimum step size that respects the tolerance parameter is not as trivial, thankfully one is described in section  of Computer aided geometric design.

![[CAGD fixed flattening step]]

## Forward differencing

![[Forward differencing]]

## Adaptative forward differencing
![[Adaptative forward differencing]]
## Hybrid forward differencing

![[Hybrid forward differencing]]

## Parabolla approximation

- Used in Gecko
- http://www.cccg.ca/proceedings/2004/36.pdf

## Raph Levien

This method deserves a real name, but short of that I'll call it by the name of its creator, Raph Levien.

Blog post: https://raphlinus.github.io/graphics/curves/2019/12/23/flatten-quadbez.html
- Only works with quadratic bézier curves. Cubic curves are first approximated with a sequence of quadratic bézier curves.
- The maths are also based on a parabolla approximation.
- Very good at distributing segments where the curvature is the most important.
- Generates few segments for a given tolerance threshold
- Pretty expensive on the CPU (lots of square roots).
- Used on the GPU in [[Vello]] and [[Forma]]

## Antigrain adaptative flattening

Blog post: https://web.archive.org/web/20190329074058/http://antigrain.com:80/research/adaptive_bezier/index.html

# Flatness criteria

As mentioned earlier

Discussion: https://comp.graphics.algorithms.narkive.com/w4co31vm/adaptive-subdivision-of-bezier-curves

## Max euclidean distance

This method is based on the distance between the baseline and the control points. In addition for cubic curves before the first subdivision we have to ensure that the control points "go in the direction of the baseline" (TODO: explain better)

Cubic bézier curves:
```Rust
let baseline = curve.to - curve.from;
let v1 = curve.ctrl1 - curve.from;
let v2 = curve.ctrl2 - curve.from;
let c1 = baseline.cross(v1);
let c2 = baseline.cross(v2);
let d1 = (c1 * c1) / baseline.square_length();
let d2 = (c2 * c2) / baseline.square_length();
// Different factor depending on whether or not the control points
// are on the same side of the baseline.
let factor = if (c1 * c2) > 0.0 { 3.0 / 4.0 } else { 4.0 / 9.0 };
let f2 = factor * factor;
let threshold = tolerance * tolerance;
return d1 * f2 <= threshold
	&& d2 * f2 <= threshold
	&& baseline.dot(v1) >= S::ZERO
	&& baseline.dot(curve.ctrl2 - curve.to) <= S::ZERO;
```

## Blend2d

Taken from https://blend2d.com/research/simplify_and_offset_bezier_curves.pdf p. 26

For cubic bézier curves:
$dmax = \frac{1}{8} * ||2 * from - 3 * ctrl1 + to|| + \frac{1}{8} * ||from - 3 * ctrl2 + 2 * to||$

## Ideal control point arrangment

The flatness criterion from Microsoft's HFD flattener implementation in WPF.

A cheap and more reliable flatness test based on the observation that when the curve is a uniform speed straight line from end to end, the control points are evenly spaced from beginning to end. Therefore, our measure of how far we deviate from that ideal uses distance of the middle controls, not from the line itself, but from their ideal *arrangement*. Point 2 should be half-way between points 1 and 3; point 3 should be half-way  
between points 2 and 4.

Performance can be improved by eliminating the square roots in the distance tests, retaining the Euclidean metric, but the taxicab metric is faster, and also safe.

code:
```Rust
let v212 = curve.ctrl1 - curve.from;
let v214 = curve.ctrl2 - curve.ctrl1;
let v216 = curve.to - curve.ctrl2;
let v218 = v214 - v212;
let v220 = v216 - v214;
return taxicab_norm(&v218).max(taxicab_norm(&v220)) <= tolerance;
```

This flatness criterion is tends to generate a lot more edges, but is very fast to evaluate. The HFD implementation uses a derivation of this that can be computed locally without having a full representation of the remaining curve.

## AGG

```Rust
let baseline = curve.to - curve.from;
let c1 = baseline.cross(curve.ctrl1 - curve.to);
let c2 = baseline.cross(curve.ctrl2 - curve.to);
return (c1 + c2) * (c1 + c2) <= tol * tol * baseline.square_length();
```

## CAGD

See the flattening step described earlier in the CAGD section.

# Comparison

![[Flattening comparison]]