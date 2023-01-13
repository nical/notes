
- Iterative process
- Cheap to compute on the CPU
- Bad at minimizing the number of edges, as we typically pick a constant step `dt`.
- Typically used in sweep/scanline rasterizers where we only care about the intersection and slop of the curve for a particular row and the number of edges does not have a large effect on performance
- best explaination I found so far in https://www.scratchapixel.com/lessons/geometry/bezier-curve-rendering-utah-teapot/fast-forward-differencing

# Cubic bézier curves

Expressing the cubic bézier curve in polynomial form:

$P(t) = A * t^3 + B * t^2 + C * t + D$

with:
$A = -from + 3 * ctrl1 - 3 * ctrl2 + to$
$B = 3 * from - 6 * ctrl1 + 3 * ctrl2$
$C = -3 * from + 3 * ctrl1$
$D = from$

This form is called a power series in one variable `t`, it can be differenciated term by term:

$P'(t) = 3 * A * t^2  +  2 * B * t  +  C$
$P''(t) = 6 * A * t  +  2 * B$
$P'''(t) = 6 * A$
$P''''(t) = 0$


Note that for the fourth derivative is zero since we have a cubic polynomial, and the third one is constant.
Applying this method to the quadratic bézier curve will yiel fewer terms with the third derivative being nil.

Taylor series of the cubic bézier curve polynomial:

$P(t) = P(t0) + P'(t0) * (t - t0) + P''(t0) * \frac{(t - t0)^2}{2!} + (P'''(t0)) * \frac{(t - t0)^3}{3!}$

We don't need to add more terms since the derivatives starting at the fourth are null.

We are more interested in the Taylor series expressed in the following form:
```
P(t + dt) = P(t) + P'(t) * dt + P''(t) * dt^2 / 2 + P'''(t) * dt^3 / 6
            ----   ----------   -------------       --------------
            pos    FD1          FD2                 FD3
```
Parts of the equation have been labeled (`pos`, `FD1`, `FD2` and `FD3`) so that we can easily recognize them later.

Let's apply the taylor decomposition to the first derivative $P'(t)$:

$P'(t + dt) = P'(t) + P''(t) * dt + P'''(t) * dt^2 / 2$

Multiplying each side by `dt`:
```
P'(t + dt) * dt = P'(t) * dt + P''(t) * dt^2 + P'''(t) * dt^3/2
---------------   ----------   -------------   --------------
FD1_next          FD1          FD2             FD3
```
Notice how we find our terms `FD1`, `FD2` and `FD3` in the equation that allows us to compute the next iteration of `FD1`.

Let's give the same treatment to the second derivative:

$P''(t + dt) = P''(t) + P'''(t) * dt$

This time we multiply each side by `dt^2`:
```
P''(t + dt) * dt^2 = P''(t) * dt^2 + P'''(t) * dt^3
------------------   -------------   --------------
FD2_next             FD2             FD3
```

There is a pattern: for each term of the taylor decomposition, the next iteration can becomputed by adding somethig to the previous iteration, which only depends on smaller terms on the right.

We established that the third derivative is constant so we can put all of this together:

At each step:
```
pos += FD1 + FD2 / 2 + FD3 / 6
FD1 += FD2 + FD3 / 2
FD2 += FD3
```
with initial values:
```
pos = from
FD1 = C * dt = (-3 * from + 3 * ctrl1) * dt
FD2 = 2 * B * dt^2 = (6 * from - 12 * ctrl1 + 6 * ctrl2) * dt2^2
FD3 = 6 * A * dt^3 = (6 * -from + 18 * ctrl1 -18 * ctrl2 + 18 * to) * dt^3
```

# Quadratic bézier curves

With quadratic bézier curves, the the second derivative is constant and the third is null.

polynomial form:

$$P(t) = A * t^2 + B * t + C$$

with:

$A = from - 2 * ctrl + to$
$B = 2 * ctrl - 2 * from$
$C = from$

Derivatives:

$P'(t) = 2 * A * t + B$
$P''(t) = 2 * A$
$P'''(t) = 0$

`FD3` is null, `FD2` is constant.

At each step:
```
pos += FD1 + FD2 / 2
FD1 += FD2
```

with initial values:
```
pos = from
FD1 = P'(0) * dt = (2 * ctrl - 2 * from) * dt
FD2 = P''(0) * dt^2 = (2 * from - 4 * ctrl + 2 * to) * dt^2
```
