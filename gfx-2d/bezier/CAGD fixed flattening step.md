
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
