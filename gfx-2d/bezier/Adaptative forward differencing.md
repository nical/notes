---
aliases: AFD
---

A [[Bézier flattening]] algorithm (the approach works with any type of polynomial).

One thing to note with this method is that it was clearly designed to have an efficient implementation on top of integer arithmetic.

See also: https://www.eecs.umich.edu/courses/eecs598-1/lien87.pdf

Implementation at https://github.com/wtholliday/nanovg/commit/6d08c306904344c06a0e63e386ae9090fa2bb73e

(The following is taken from the HFD patent which has some typos so hopefully I got it right)

The AFD basis formula is:

$f(t) = a_0 * A_0(t) + a_1 * A_1(t) + ... + a_k * A_k(t)$

With:

$A_k(t) = \frac{t-k+1}{k} * A_{k-1}(t)$
$A_0(t) = 1$

Specifically for cubic bézier curves:

$f(t) = a_0 + a_1 * A_1(t) + a_2 * A_2(t) + a_3 * A_3(t)$
$A_0(t) = 1$
$A_1(t) = t$
$A_2(t) = \frac{t * (t - 1)}{2}$
$A_3(t) = \frac{t * (t - 1) * (t - 2)}{6}$

At each step:

$f(t+dt) = (a_0 + a_1) * A_0(t) + (a_1 + a_2) * A_1(t) + (a_2 + a_3) * A_2(t) + a_3 * A_3(t)$

The following comes from the original AFD paper (I changed the notation, hopefully without messing it up):

# Matrix notation

$L$ is the linear substitution $\frac{t}{2}$ (left side of the subdivision)
$R$ is the linear substitution $\frac{t + 1}{2}$ (right side of the subdivision we don't really need to use it in practice)
$L^{-1}$ is the linear substitution $2*t$ (doubling the step size)
$E$ is the linear subsitution $t+1$ (next iteartion without changing the step size)

$$
E = \begin{bmatrix}
1 & 1 & 0 & 0 \\
0 & 1 & 1 & 0 \\
0 & 0 & 1 & 1 \\
0 & 0 & 0 & 1 \\
\end{bmatrix}
$$
$$
L = \begin{bmatrix}
1 & 1 & 0 & 0 \\
0 & \frac{1}{2} & \frac{-1}{8} & \frac{1}{16} \\
0 & 0 & \frac{1}{4} & \frac{-1}{8} \\
0 & 0 & 0 & \frac{1}{8} \\
\end{bmatrix}
$$
$$
L^{-1} = \begin{bmatrix}
1 & 1 & 0 & 0 \\
0 & 2 & 1 & 0 \\
0 & 0 & 4 & 4 \\
0 & 0 & 0 & 8 \\
\end{bmatrix}

$$
# Steps

Note: the $f'$ notation here refers to the next step, not derivatives.

## Having the step 

To reduce $dt$ by half, we transform the cubic function by applying the $L$ matrix:

$f'(t) = f(\frac{t}{2}) = a_3' * A_3 + a_2' * A_2 + a_1' * A_1 + a_0' * A_0$

With:

$a_3' = \frac{1}{8} * a_3$
$a_2' = \frac{1}{4} * a_2 - \frac{1}{8} * a_3$
$a_1' = \frac{1}{2} * a_1 - \frac{1}{8} * a_2 + \frac{1}{16} a_3$
$a_0' = a_0$

## Doubling the step

To double $dt$ we have:

$a'_3 = 8 * a_3$
$a_2' = 4 * a_2 + 4 * a_3$
$a_1' = 2 * a_1 + a_2$
$a_0' = a_0$

## At constant step

When the step size does not change:

$a_3'= a_3$
$a_2' = a_2 + a_3$
$a_1' = a_1 + a_2$
$a_0' = a_0 + a_1$

# Initial parametrisation

Using the coefficient of the curve in power basis
$$
f(t) = P_0 + P_1*t + P_2*t^2 + P_3*t^3;
$$

$a_3 = P_0$
$a_2 = P_1 + P_2 + P_3$
$a_1 = 6 * P_1 + 2 * P_2$
$a_0 = 6 * P_1$

# Flatteness criterion

#TODO Also missing in the paper?

