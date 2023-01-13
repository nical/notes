Bézier curve flattening tends to have a big impact on performance, various approaches

# Algorithms

## Recursive subdivision

- Very simple algorithm.
- Not terrible but not great either at minimizing the number of edges.

## Forward differencing

![[Bézier forward differencing]]

## Hybrid forward differencing

- Patent https://patents.google.com/patent/US5367617 expired in 2012-07-02

## Parabolla approximation

- Used in Gecko
- http://www.cccg.ca/proceedings/2004/36.pdf

## Raph Levien

Blog post: https://raphlinus.github.io/graphics/curves/2019/12/23/flatten-quadbez.html
- Very good at distributing segments where the curvature is the most important
- Generates few segments for a given tolerance threshold
- Pretty expensive on the CPU (lots of square roots)
- Used on the GPU in [[Vello]]

## Antigrain adaptative flattening

Blog post: https://web.archive.org/web/20190329074058/http://antigrain.com:80/research/adaptive_bezier/index.html

# Flatness criteria

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

## Hybrid forward differencing (HFD)

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

Flatness criterion from section 10.6 of [[Computer Aided Geometric Design.pdf]] ([link](https://scholarsarchive.byu.edu/cgi/viewcontent.cgi?article=1000&context=facpub#section.10.6))

```
error(t) = ||P(t) - Line(t)|| = L * dt^2 / 8
```

The number of segments required when splitting the curve at regular `t` intervals given a certain `tolerance` threshold:
$n_{segments} = \sqrt{\frac{||L||}{8 * tolerance}}$

Or the number of recusive subdivisions: $n_{subdivs} = log2(n_{segments})$

with: $L = n(n - 1) * max_{0 ≤ i < n-2}(Curve_{i+2} - Curve_{i} + Curve_{i})$

For quadratic béziers (n = 2): $L = 2 * from - 4 * ctrl + 2 * to$
And cubic béziers (n = 3): $L = 6 * max(from - 2 * ctrl1 + ctrl2, ctrl1 - 2 * ctrl2 + to)$

# Comparison

Perf score (measured on Mac m1 max) lower is better:

Cubic bézier curves:
| Method    | Perf score | edge count | edges (vs rec) |
| --------- | ----------:| ----------:| --------------:|
| recursive |       27,5 |       7934 |           1.00 |
| rec_hfd   |       82.2 |     211233 |           2.66 |
| rec_agg   |       49.4 |      13813 |           1.74 |
| CAGD      |        9.5 |      16417 |           2.07 |
| fwd-diff  |        9.7 |      16417 |           2.07 |
| hfd       |       15.7 |      19019 |           1.40 |
| levien    |       34.2 |      10157 |           1.24 |
| linear    |       30.9 |       8034 |           1.01 |
| linear2   |       23.5 |       8035 |           1.01 |
| pa        |       45.0 |       6813 |           0.86 |

Quadratic bézier curves:
| Method | Perf score | edge count | edges (vs recursive)|
|--------|------------|------------|-------------------|
| recusive| 13.2 | 7114  | 1.00 |
| linear2 | 10.4 | 7007  | 0.98 |
| fwd-diff|  5.8 | 14442 | 2.03 |
| cagd    |  6.3 | 14442 | 2.03 |
| levien  |  8.7 | 6176  | 0.87 |

### Performance

Lower is better
![[flatten-perf-cubic-x86.svg]]
![Line Chart](flatten-perf-cubic-m1.svg)

![[flatten-perf-quadratic-x86.svg]]
![Line Chart](flatten-perf-quadratic-m1.svg)

### Edge count

Lower is better
![[edges-cubic.png]]
![[edges-quadratic.png]]

Cubic bézier curves:

|tolerance | 0.01| 0.025| 0.05| 0.1| 0.15| 0.2| 0.25| 0.3| 0.4| 0.7| 1|
|----------| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:|
|recursive | 48599 | 31379 | 22026 | 16001 | 12982 | 11170 | 10120 | 9354 | 8198 | 6226 | 5329 |
|rec_hfd   | 102948 | 65593 | 47681 | 34829 | 28842 | 24987 | 22259 | 20350 | 17661 | 13408 | 11403 |
|rec_agg   | 88346 | 54968 | 39697 | 28462 | 23287 | 20322 | 18079 | 16262 | 13948 | 10339 | 8477 |
|fwd-diff  | 80422 | 51199 | 36487 | 26070 | 21460 | 18682 | 16842 | 15419 | 13482 | 10411 | 8873 |
|hfd       | 90911 | 58036 | 41569 | 29922 | 24843 | 21646 | 19431 | 17777 | 15673 | 12285 | 10377 |
|pa        | 34988 | 22764 | 16616 | 12273 | 10329 | 9188 | 8422 | 7850 | 7102 | 5813 | 5278 |
|levien    | 71630 | 41250 | 27510 | 18549 | 14730 | 12570 | 11149 | 10121 | 8676 | 6576 | 5505 |
|linear    | 47795 | 30406 | 21550 | 15306 | 12467 | 10876 | 9809 | 8957 | 7829 | 5942 | 5147 |
|linear2   | 47866 | 30480 | 21582 | 15398 | 12510 | 10904 | 9821 | 8970 | 7835 | 5944 | 5151 |
|cagd      | 80422 | 51199 | 36487 | 26070 | 21460 | 18682 | 16842 | 15419 | 13482 | 10411 | 8873 |

  

Quadratic bézier curves:

|tolerance | 0.01| 0.025| 0.05| 0.1| 0.15| 0.2| 0.25| 0.3| 0.4| 0.7| 1|
|----------| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:| -----:|
|recursive | 48611 | 30764 | 22212 | 15762 | 13117 | 11519 | 10406 | 9547 | 8420 | 6893 | 6016 |
|fwd-diff  | 70558 | 45416 | 32772 | 23852 | 19859 | 17537 | 15917 | 14640 | 13074 | 10316 | 8989 |
|levien    | 36291 | 23934 | 17647 | 13269 | 11208 | 10060 | 9182 | 8624 | 7810 | 6473 | 5798 |
|linear2   | 46321 | 29228 | 20907 | 15016 | 12432 | 11034 | 10068 | 9206 | 8254 | 6723 | 5919 |
|cagd      | 70558 | 45416 | 32772 | 23852 | 19859 | 17537 | 15917 | 14640 | 13074 | 10316 | 8989 |

Notes:
- The parabola approximation implementation is probably buggy.
- Raph Levien's method is very competitive with quadratic curves but not so much with cubics. The `t` values are also mapped incorrectly back to the cubic curve (bug in lyon's implementation).
- cairo's flattening implementation is a simple recursion using distance between the control points and the baseline as the flattening criterion (similar to `recursive`). See `_cairo_spline_decompose_into`


Color palette used for the perf graphs:
```
recursive acacac
levien e85b0a
linear2 268cf2
cagd b7db0d
fwd-diff 854be5
hfd ed7de3
linear 49aab8
pa e4be08
```