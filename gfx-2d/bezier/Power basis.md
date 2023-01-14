# Polynomial form

$$
f(t) = P_0 + P_1*t + P_2*t^2 + P_3*t^3;
$$

For cubic béziers:
```C
P0 = from;
P1 = -3 * from + 3 * ctrl1;
P2 = 3 * from - 6 * ctrl1 + 3 * ctrl2;
P3 = -from + 3 * ctrl1 -3 * ctrl2 + to;
```
For quadratic béziers:
```C
P0 = from;
P1 = 2 * ctrl - 2 * from;
P2 = from + to - 2 * ctrl;
P3 = 0;
```


# Horner's method

Sampling a cubic bézier curve in polynomial form can be written as:
```C
P = P0;
term = t;
P += term * P1;
term *= t;
P += term * P2;
term *= t;
P += term * P3;
```
This requires fewer instructions than evaluating the usual bézier formula if we already have the bézier curve in polynomial form.

