Using the power basis vector of a sampling a bézier curve at $t$ can be written in the following matrix forms:

For cubic bézier curves:
$$
\begin{bmatrix}
from & ctrl1 & ctr2 & to
\end{bmatrix}

\begin{bmatrix}
1 & -3 &  3 & -1\\
0 &  3 & -6 &  3\\
0 &  0 &  3 & -3\\
0 &  0 &  0 &  1\\
\end{bmatrix}

\begin{bmatrix} 1 \\ t \\ t^2 \\ t^3\end{bmatrix}

$$

For quadratic bézier curves:
$$
\begin{bmatrix}
from & ctrl & to
\end{bmatrix}

\begin{bmatrix}
1 & -2 &  1\\
0 &  2 & -2\\
0 &  0 &  1\\
\end{bmatrix}

\begin{bmatrix} 1 \\ t \\ t^2 \end{bmatrix}

$$

There is an infinity of bases, some of which are more convenient or useful than others. See also:
 - AFD basis in [[Adaptative forward differencing]].
 - HFD basis in [[Hybrid forward differencing]].

Why is is the matrix form useful? #TODO 
- The matrix representation is more general (can express more types of curves).
- Reasoning in matrix terms maybe gives us access to interesting properties of matrices?
- To leverage hardware or infrastructure that is specialized for matrix multiplication?
