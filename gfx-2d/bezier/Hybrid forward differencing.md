---
aliases: HFD
---

A [[Bézier flattening]] algorithm.

Patent https://patents.google.com/patent/US5367617 expired since 2012-07-02

Very similar to [[adaptative forward differencing]], the key differences are:
 - The math is epxressed in a different basis.
 - The flatness criterion which the HFD basis allows.

The HFD scaled error basis for a cubic bézier polynomial is written in the form:

$f(t) = e_0 * E_0(t) + e_1 * E_1(t) + e_2 * E_2(t) + e_3 * E_3(t)$

With:

$E_0(t) = A_0(t) = 1$
$E_1(t) = A_1(t) = t$
$E_2(t) = A_2(t) + A_3(t) = \frac{t * (t^2 - 1)}{6}$
$E_3(t) = A_3(t) = \frac{t * (t - 1) * (t - 2)}{6}$

Where $A_0(t)$, $A_1(t)$, $A_2(t)$ and $A_3(t)$ are the [[Forward differencing]] basis.

It results in the forward step operator:

$e_0 = from$
$e_1 = to - from$
$e_2 = 6 * (to - 2*ctrl2 + ctrl1)$
$e_3 = 6 * (ctrl2 - 2 * ctrl1 + from)$

$e_2$ and $e_3$ are 6 times vectors v1 and v2 below. Their magnitude is compared to the tolerance threshold.

```
crtl1         ctrl2
    x---------x
   / \v1    /  \
  /    \  /v2   \
 /      /\       \
x     v    v      x
from              to
```

## Step

```Rust
position += e1;
let tmp = e2;
e1 += e2;
e2 += e2 - e3;
e3 = tmp;
```
After each iteration, check the flatness criterion and potentially halve or double the step until an optimal step is found.

## Halving the step

```Rust
e2 = (e2 + e3) / 8;
e1 = (e1 - e2) / 2;
e3 = e3 / 4;
```

## Doubling the step

```Rust
e1 = e1 * 2 + e2;
e3 = e3 * 4;
e2 = (e2 * 2 - e3) * 4;
```

## Initial parametrisation

First initialize with step `dt = 1` (see the earlier forward step operator): 
```Rust
e0 = from;
e1 = to - from;
e2 = 6 * (to - 2*ctrl2 + ctrl1);
e3 = 6 * (ctrl2 - 2 * ctrl1 + from);
```
Then halve the step until the flattening criterion is met.

## Flatness criterion

A cheap flatness test based on the observation that when the curve is a uniform speed straight line from end to end, the control points are evenly spaced from beginning to end. Therefore, our measure of how far we deviate from that ideal uses distance of the middle controls, not from the line itself, but from their ideal *arrangement*. `ctrl1` should be half-way between `from` and `ctrl2` and `ctrl2` should be half-way  between `ctrl1` and `to`.

The deviation of `ctrl1` and `ctrl2` from their ideal arrangment are:

$dev1 = \frac{from + ctrl2}{2} - ctrl1 = \frac{1}{12} * e_3$
$dev2 = \frac{ctrl1 + to}{2} - ctrl2 = \frac{1}{12} * e_2$

In which we recognize the earlier formulaitons of $e_2$ and $e_3$.

Note that this flatness criterion incorporates the notion of uniform speed. That's probably why it is faring so much worse than criterions that only take the distance from the curve in consideration in terms of the number of generated edges.

Performance can be improved by eliminating the square roots in the distance tests, retaining the Euclidean metric, but the taxicab metric is faster, and also safe.

code:
```Rust
let v212 = curve.ctrl1 - curve.from;
let v214 = curve.ctrl2 - curve.ctrl1;
let v216 = curve.to - curve.ctrl2;
let v218 = v214 - v212; // dev1 * 2
let v220 = v216 - v214; // dev2 * 2
return taxicab_norm(&v218).max(taxicab_norm(&v220)) <= tolerance;
```
Or with euclidean distance:
```Rust
let sq_dev = v218.square_legnth().max(v220.square_length());
return sq_dev <= tolerance * tolerance * 4 * k * k;
```
Since  `taxicab_norm(v) <= sqrt2(2) * norm(v)` , `k = sqrt(2)` should work.

This flatness criterion tends to generate a lot more edges than the other criterions I have tried, but is very fast to evaluate. The HFD implementation uses a derivation of this that can be computed locally without having a full representation of the remaining curve.
