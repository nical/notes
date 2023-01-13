# In layman's terms

For a lot of cases it's sufficient to think of a monoid as "an associative transformation f(a,b) -> c with two inputs and an output of the same type". It's a bit more than that but I think that a lot of the interesting things (for programming purposes anyway) comme off of that.

The other property of a monoid is that it has an identity or "neutral" element `f(a, identity) == a`.

# Mathematical definition

A monoid is a set `S` equipped with an associative binary operation `op(a: S, b: S) -> S` and an identity element.
 - It is *closed*: the inputs and outputs of the opearion belong to the set (a magma).
 - It is *associative*: `op(a, op(b, c)) == op(op(a, b), c)` (a semigroup).
 - It has an identity element.

## Examples

- Non-negative integers with additions form an monoid (identity being `0`).
- Real numbers with multiplications form a monoid (identity being `1`).

# In computer science

## Examples

- Strings and concatenation

## Interesting properties of monoids

The fact that a monoid is closed (consumes and produces data of the same type), lends itself to composing operations, but properties like associativity is what make it possible to reorganise these compositions in interesting ways.

A common pattern is for a sequence of elements of a monoid to be accumulated/folded to produce a final value. For instance a [[Prefix sum]]. This pattern may be expressed by a monoid operation.

The associativity of monoid operations ensures that some higher level operations that are built upon them can leverage parallelism. For example computing the sum of elements `e0, e1, e2, e3, e4, e5, e6, e7` could be expressed in a very sequential way:
```
add(
  add(
    add(
      add(
        add(
          add(
            add(e0,e1),
            e2)
          e3),
        e4),
      e5),
    e6),
  e7,
)
```
However thanks to associativity the same result can also be achieved via:
```
add(             // e01234567
  add(           // e0123
    add(e0, e1), // e01
    add(e2, e3), // e23
  ),
  add(           // e4567
    add(e4, e5), // e45
    add(e6, e7), // e67
  ),
)
```
Where some elements can be computed in parallel (e01, e23, e34, e45, e67 in a first parallel step, then e0123 and e4567, etc.) 

- MapReduce
- Fold

#TODO

## Why bother ?

It's fairly evident that summing `n` numbers can be broken up into `log(n)` parallel passes without thinking in mathematical terms. However, for more complicated problems it might be harder to see it. For these it is sufficient to express the problem in terms of a scan and a monoid operation to prove that the parallel version is correct.

The trick is then to find how the data and the transformation can be expressed.

The data does not have to be in the same for as the input and output of the problem. For example, parenthese matching which is about finding the offset of the openning parenthese of the current `()` -delimited block (which cam be nested) for each element in a steam.
 - The data of the monoid can be expressed as a command: `{ pop: int, push: sequence<int> }`:
	 - `(` at position `i` is  the command `{ pop: 0, push [i] }`.
	 - `)` at any position is  the command `{ pop 1, push [] }`.
	 - The identity command is `{ pop 0, push [] }` (for any character in the stream that isnt `(` or `)`).
	 - The operation that combines commands is pretty straightfoward (see linked blog post).
 - See the second blog post from Raph Levien linked below for explanations, the point being that the data of the monoid is not necessarily something as straightforward as a character in the stream.

Applications of this include [parallel parsing of context-free languages](https://www.cambridge.org/core/journals/journal-of-functional-programming/article/efficient-parallel-and-incremental-parsing-of-practical-contextfree-languages/4D620F0BFADE2B588F854AAAEA252F5C).

## Raph Levien's work

Raph loves to describe parts of the piet/vello work in terms of monoids.
 - https://raphlinus.github.io/gpu/2020/09/05/stack-monoid.html
 - https://raphlinus.github.io/gpu/2021/05/13/stack-monoid-revisited.html

A lot of it has to do with the need to do [[Prefix sum]]s and other types of parallel scans (See [[parallel-scan.pdf]]) on the GPU in [[Vello]].
