See also [What every programmer should know about floating point arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)


Detecting potentially problematic cases:
- test whether the values calculated are sufficiently far from zero that they may be used without any possibility of error.

# problem with fixed epsilons 

They only work when the values are within a certain range, the greater the number the greater the epsilon should be 

# comparing bit patterns 

IEE754 floats are designed to maintain their order when their bit patterns are interpreted as integers 

# Using rational numbers 

https://en.m.wikipedia.org/wiki/Rational_number 
Use [arbitrary precision arithmetic](https://en.m.wikipedia.org/wiki/Arbitrary-precision_arithmetic)

# Fixed-point representation

Pros:
- Implemented on top of simple integer arithmetic which used to be a fair bit faster than float arithmetic on older CPUs.
- precision is evenly distributed over the coordinate space.
- Easier to reason about precision.
Cons:
- limited precision
- easy to run into overflows

Integer and fixed point number arithmetic can also have different kinds of precision issues when performing divisions and multiplications.

The problem with fixed point and integer coordinates is that it is very easy to run into overflows because they can describe a much smaller range than floating point number. There's a trade-off to make between how much precision we want in the fractional part and the size of the range. Currently the tessellator works with 32 bit fixed point precision numbers with 16 bits for fractional part, while the line segment intersection computation is implemented with f64 floating point numbers. There's been experiments with various fixed point precision math and just integer arithmetic, but it's quite hard to not run into overflows with the them when postponing division (otherwise the precision loss is worse than float math so why bother).

The challenge, when trying to not lose any precision, is that every time the internal representations of two fixed point numbers are multiplied, we need to potentially double the amount of bits required to hold the result, which grows quickly over the 64 bits we are comfortable working with. So postponing all divisions to the end of a long series of computation is not always a viable option.

For reference Skia's tessellator stores positions using 16.16 fixed point numbers and uses doubles to compute edge intersections. Since the resulting intersections are not exact, there are corrective measures that are taken later in the algorithm to correct cases where the imprecision cause some invariants of the algorithm to be broken. This may turn out to be the best compromise (unfortunately).

# Citardauq formula

TODO