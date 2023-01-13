- Pomax's bézier primer: https://pomax.github.io/bezierinfo/
- Freya's continuity of splines: https://youtu.be/jvPPXbo87ds


See also:

 - [[Bézier flattening]]
 - [[Bézier intersection]]

# Polynomial form


```C
P(t) = A0 + A1*t + A2*t^2 + A3*t^3;
```

For cubic béziers:
```C
A0 = from;
A1 = -3 * from + 3 * ctrl1;
A2 = 3 * from - 6 * ctrl1 + 3 * ctrl2;
A3 = -from + 3 * ctrl1 -3 * ctrl2 + to;
```
For quadratic béziers:
```C
A0 = from;
A1 = 2 * ctrl - 2 * from;
A2 = from + to - 2 * ctrl;
A3 = 0;
```



# Horner's rule

Sampling a cubic bézier curve in polynomial form can be written as:
```C
P = A0;
term = t;
P += term * A1;
term *= t;
P += term * A2;
term *= t;
P += term * A3;
```
This requires fewer instructions than evaluating the usual bézier formula if we already have the bézier curve in polynomial form.