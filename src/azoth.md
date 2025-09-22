# `azoth` Namespace

## `azoth.Cancellation_Token`

Cancellation tokens are used to manage cancellation. A function that supports cancellation should
take a `Cancellation_Token` as the last parameter as a context parameter. The `throw_if_cancelled()`
method should be used to check whether cancellation has been requested and throw `Cancelled` if it
has. Cancellation tokens internally are built around two mechanisms for cancellation. A deadline
being reached on a monotonic clock and arbitrary cancellation predicates.

## `azoth.Exception`

Exception is a base class that provides basic functionality for exceptions. Note that a type does
not need to inherit from `Exception` to be throwable. It is just a base class that it is recommended
that all reference types meant for throwing inherit from.

Exception provides two primary benefits. First, it includes an inner exception for use when
catching an exception of one type and wrapping it to throw an exception of another type. Second, it
provides an exception message that interacts well with structured logging. Exception messages may
contain references to properties of the exception. These messages play well with structured logging
so that when an exception is logged, there is a distinction between the template of the exception
message and the actual message with values filled in. Also, exception properties referenced in the
message can be logged as associated data.

Note that `Exception` does not have any associated stack trace information like it does in languages
such as C# or Java.

## `azoth.Iterable`

## `azoth.Iterator`

### Enumerate

The standard library provides an `enumerate()` method for counting. This provides the index of each
value in the iterator sequence.

```azoth
foreach #(i, x) in (5..10).enumerate()
{
    console.write_line("i = \(i), x = \(x)");
}
```

Outputs:

```console
i = 0, x = 5
i = 1, x = 6
i = 2, x = 7
i = 3, x = 8
i = 4, x = 9
i = 5, x = 10
```

## Ranges

*Note:* The exact range API has not been determined. There may need to be a distinction between
ranges of continuous types and ranges of discrete types. See Swift 3 ranges and `Strideable` for an
example https://oleb.net/blog/2016/09/swift-3-ranges/.

Any type `T where T: Integral` implements the range operator `X..Y` so that is returns a `Range[T,
1]` inclusive of `X` and `Y`. If `X > Y` then the range returned is the empty range. Note the type
of `Range` is `Range[T, Step:T=1] where T: number`. This needs to fit in with an `Interval` type
defined in `azoth.math`. To construct a range exclusive of the end value use the `..<` operator.
Likewise the `<..` and `<..<` operators are used to construct ranges exclusive of the start and
exclusive of both the start and end respectively.

Related types:

* ReverseRange - instead of building negative step into range
* RangeFrom - range only bounded below
* RangeTo - range only bounded above
* RangeFull/FullRange - unbounded range
* InclusiveRange - range inclusive of the end value
* StepBy? - a way to represent stepping ranges
* Interval
* Distribution used by random etc.

## Abort

`ABORT(message: string) -> never` can be used to cause a program abort.

## Unreachable

`UNREACHABLE(message: string) -> never` or `UNREACHABLE() -> never` can be used to to indicate a
point in the program that ought to be logically unreachable. If at execution time, it is reached
then a program abort happens.
