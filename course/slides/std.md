# Handling Failure

## Motivation

Remember the `lighten` method:

~~~ scala
def lighten = Barbell(load - 10, length - 20)
~~~

> - What happens if the `load` or the `lenth` becomes zero or less?
> - Do you want `lighten` to be **defined** for **all** values of `Barbell`?

## `Option`

A way to model the fact that a given barbell may not have a lighter barbell is to use the `Option` type:

~~~ scala
def lighten: Option[Barbell] =
  if (load <= 15 || length <= 100) None
  else Some(Barbell(load - 10, length - 20))
~~~

- The standard library defines the type `Option[A]` that models an optional value of type `A`
- An `Option[A]` value can either be:
    - `Some(a)`
    - `None`

## Exercise

- Add a `smaller` method to the `Mat` type:

~~~ scala
def smaller: Option[Mat] = ???
~~~

## Common Patterns with Optional Values

`Option` values can be manipulated using pattern matching:

~~~ scala
def whatIsIn(maybeBarbell: Option[Barbell]): String =
  maybeBarbell match {
    case Some(barbell) => "there is a barbell"
    case None => "there is no barbell"
  }
~~~

## Common Patterns With Optional Values

Use `map` to transform a successful value into another successful value, ignoring the `None` case:

```scala
def maybeWidth(maybeMat: Option[Mat]): Option[Int] =
  maybeMat.map(mat => mat.width)
```

## Common Patterns With Optional Values (2)

Use `filter` to turn a successful value into a failure if it does not satisfy a given predicate:

```scala
def keepHugeMats(maybeMat: Option[Mat]): Option[Mat] =
  maybeMat.filter(mat => mat.width > 100 && mat.length > 200)
```

## Common Patterns With Optional Values (3)

Use `flatMap` to transform a successful value into an optional value:

```scala
def smallerSmaller(mat: Mat): Option[Mat] =
  mat.smaller.flatMap(smallerMat => smallerMat.smaller)
```

## Sequencing Computations Manipulating Optional Values

`flatMap` and `map` are used to sequence computations operating on optional values:

```scala
def lightenLightenLoad(barbell: Barbell): Option[Int] =
  barbell.lighten.flatMap { lighterBarbell =>
    lighterBarbell.lighten.map { lighterLighterBarbell =>
      lighterLighterBarell.load
    }
  }
```

(We will see a more expressive way of expressing such computations, soon)

## `Try`

- `Try[A]` is _similar_ to `Option[A]` excepted that in case of failure it provides more information. It can either be:
  - `Success(a)`
  - `Failure(throwable)`

## Common Patterns with `Try[A]` Values

- Like `Option[A], `Try[A]` has `map`, `filter` and `flatMap`

<!--
TODO Exceptions try/catch
-->

# Standard Collections

## Standard Collections

The Scala standard library provides several types making it easier to deal with collections of elements

This section gives a slight overview of the standard collections. For more details see the [API documentation](http://scala-lang.org/api)

## `Traversable`

The most general one is `Traversable[A]`, it provides methods to iterate on the elements of a collection, to transform a collection, to filter it, and a lot more:

Method               Description
----------           -------------
`xs.foreach(f)`      Applies the function `f` to every element of `xs`
`xs ++ ys`           The concatenation of the elements of `xs` and `ys`
`xs.size`            The number of elements in `xs`
`xs.map(f)`          A collection obtained from applying `f` to every element of `xs`
`xs.flatMap(f)`      A collection obtained from applying `f` to every element of `xs` and concatenating the results
`xs.filter(p)`       A collection obtained from filtering elements of `xs` that satisfy the predicate `p`
`xs.foldleft(z)(f)`  Applies a binary operator `f` to a start value `z` and all elements of `xs`, going left to right
`xs.take(n)`         A collection containing the `n` first elements of `xs`
`xs.find(p)`         An optional value containing the first element of `xs` that satisfies `p`
`xs.headOption`      An optional value containing the first element of `xs`
`xs.tailOption`      An optional value containing the tail of `xs`

## `Iterable`

`Iterable[A]` extends `Traversable[A]`, here are some of its new members:

Method             Description
-----------        -------------
`xs.iterator`      An `Iterator[A]` over the elements of `xs`
`xs.grouped(n)`    An iterator that yields fixed-size chunks of `xs`
`xs zip ys`        An iterable of pairs of corresponding elements of `xs` and `ys`
`xs.zipWithIndex`  An iterable of pairs of elements of `xs` with their indices

## `Seq`

`Seq[A]` is an `Iterable[A]` which order of elements is kept. It adds the following members:

Method               Description
----------           ---------------
`x +: xs`            A collection with the elements of `xs` prepended with `x`
`xs :+ x`            A collection with the elements of `xs` followed by `x`
`xs.get(n)`          An optional value containing the `n`^th^ element of `xs` (0 indicates the first element)
`xs.reverse`         A collection with the elements of `xs` in reverse order
`xs.updated(n, x)`   A copy of `xs` which `n`^th^ element has been replaced by `x`
`xs.sorted`          A collection with the elements of `xs` sorted

## `Seq` (2)

`Seq` can be used with pattern matching as follows:

~~~ scala
def sum(xs: Seq[Int]): Int = xs match {
  case Nil     => 0
  case x +: xs => x + sum(xs)
}
~~~

## `List` and `Vector`

`List[A]` and `Vector[A]` are two implementations of `Seq[A]` with different performance characteristics:

- `List[A]` has more efficient `head` and `tail` implementations

- `Vector[A]` has more efficient `get` and `size` implementations

- If you want to do random access, you should use `Vector[A]`

```scala
val xs = 1 :: 2 :: 3 :: Nil // List
val ys = Vector(1, 2, 3)
```

## `Range`

`Range` is a useful implementation of `Seq[Int]` that efficiently represents a range of integer values

The simplest way to generate a range is to use `to` and `until` methods of numeric values:

```scala
(1 to 3).foreach(println) // prints “1”, “2”, “3”
(0 until 3).map(_ * 2).foreach(println) // prints “0”, “2”, “4”
```

## `Set`

A `Set[A]` is an `Iterable[A]` that contains no duplicate elements. It adds the following members:

Method               Description
-----------          ----------------
`xs + x`             A set containing the elements of `xs` and `x`
`xs - x`             A set containing the elements of `xs` without `x`
`xs.contains(x)`     Tests if `x` is contained in `xs`
`xs & ys`            Intersection of `xs` and `ys`
`xs | ys`            Union of `xs` and `ys`
`xs.subsetOf(ys)`    Tests if `xs` is a subset of `ys`

```scala
val xs = Set(1, 2, 3)
```

## `Map`

A `Map[A, B]` is an `Iterable[(A, B)]` that contains `B` values indexed by `A` values. It adds the following members:

Method               Description
-------------        ---------------
`xs.get(k)`          An optional value containing the value associated with `k`
`xs + (k -> x)`      Adds a new value `x` associated with `k`
`xs - k`             Removes the element associated with `k`
`xs.keys`            An iterable of the keys of `xs`

```scala
val xs = Map("foo" -> 42, "bar" -> 10, "baz" -> 0)
```

## Exercise

- Change the `circles` method so that it returns a collection of concentric circles:

~~~ scala
def circles(n: Int): Seq[Circle] = ???
~~~

- Note: write to different implementations: a recursive and a non-recursive one.

<!--
def circles(n: Int): Seq[Circle] = {
  val unit = Circle(25 + 15 * n)
  if (n == 1) Seq(unit)
  else circles(n - 1) :+ unit
}

def circles(count: Int) =
  (1 to count) map { n =>
    Circle(25 + 15 * n)
  }
-->

## Exercise

- Do the same for `spiral`:

~~~ scala
def spiral(n: Int): Seq[Image] = ???
~~~

<!--
def spiral(count: Int) =
  (1 to count) map { n =>
    val size = 10 + n * 2
    val dist = 50 + n * 5
    val angle = Angle.degrees((n * 36) % 360)
    Circle(size).at(dist * angle.sin, dist * angle.cos)
  }
-->

## Exercise

- Change the `stack` method to the following:

~~~ scala
def stack(images: Seq[Image]): Image = ???
~~~

- Note: write a recursive implementation and a non-recursive implementation

<!--
def stack(images: Seq[Image]): Image = images match {
  case Nil => emptyImage
  case image +: images => image on stack(images)
}

def stack(images: Seq[Image]): Image =
  images.foldLeft(emtyImage)((image, result) => image on result)
-->

## Exercise

- Change the `layout` method to the following:

~~~ scala
def layout(op: (Image, Image) => Image, images: Seq[Image]): Image = ???
~~~

<!--
def layout(op: (Image, Image) => Image, images: Seq[Image]): Image =
  images.foldLeft(emptyImage)(op)
-->