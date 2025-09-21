# Conversions

A *conversion* is a transformation of values from one type to another. Conversions can be *implicit*
or *explicit*. Implicit conversions are inserted automatically by the compiler as necessary.
Explicit conversions require an explicit conversion expression. For example, the type "`int32`" can
be implicitly converted to "`int64`", so expressions of type "`int32`" can implicitly be treated as
"`int64`". The opposite conversion, from "`int64`" to "`int32`", is explicit, so an explicit
conversion is required.

```azoth
let a: int32 = 123;
let b: int64 = a; // implicit conversion from int32 to int64
let c: int32 = b as! int32; // explicit conversion from int64 to int32
```

Generally, numeric conversions that have no risk of failure or loss of precision are implicit.
Conversions that could fail or lose data should be explicit. For example, certain numeric
conversions will not fail, but may lead to a loss of precision. These are described in [Explicit
Numeric Conversions](#explicit-numeric-conversions) and can be performed using the "`as`" operator.

Some conversions are defined by the language. Programs may also define their own user-defined
conversions.

Note that when a subtyping relationship exists between two types, then no conversion is necessary.

**TODO:** add handling of literal types (e.g. `int[V]`).

## Standard Conversions

A standard conversion is a built-in conversion that is not an optional conversion. That is,
user-defined conversions and conversions from `V` to `V?` are not standard conversions.

## Implicit Conversions

The following conversions are implicit conversions:

* Implicit numeric conversions
* Implicit optional conversions
* Implicit lifted conversions
* User-defined implicit conversions

The "`as`" operator can be used to explicitly cause an implicit conversion. Since an implicit
conversion cannot fail, the `as!` and `as?` operators cannot be used with implicit conversions.

### Multiple Conversions

An implicit conversion without a user-defined conversion will only ever involve one standard
conversion but may have an arbitrary number of optional conversions applied.

An implicit conversion with a user-defined conversion will only ever involve one user defined
conversion. It may have one standard conversion before the user defined conversion and one after. In
addition, it may have an arbitrary number of optional conversions applied.

### Implicit Numeric Conversions

The implicit numeric conversions are those conversions between simple numeric types that can be
safely performed with no loss of precision.

The implicit numeric conversions are:

| From      | To                                                                                                            |
| --------- | ------------------------------------------------------------------------------------------------------------- |
| `int`     | (none)                                                                                                        |
| `uint`    | `int`                                                                                                         |
| `int8`    | `nint`, `int16`, `int32`, `int64`, `int`, `float32`, `float64`                                                |
| `byte`    | `nint`, `nuint`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`, `float32`, `float64` |
| `int16`   | `nint`, `int32`, `int64`, `int`, `float32`, `float64`                                                         |
| `uint16`  | `nuint`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`, `float32`, `float64`                            |
| `int32`   | `int64`, `int`, `float64`                                                                                     |
| `uint32`  | `int64`, `int`, `uint`,`uint64`, `float64`                                                                    |
| `int64`   | `int`                                                                                                         |
| `uint64`  | `int`, `uint`                                                                                                 |
| `nint`    | `int`                                                                                                         |
| `nuint`   | `uint`, `int`                                                                                                 |
| `size`    | `nuint`, `uint`, `int`,                                                                                       |
| `offset`  | `nint`, `int`                                                                                                 |
| `float32` | `float64`                                                                                                     |

Note that conversions to native ints (i.e. `nint` and `nuint`) are possible because they are assumed
to be at least 16-bit. Conversions to `size` and `offset` are not supported because `size` is
further restricted and so only `byte` could be converted to it. Also, it maintains the separation of
logic involving indexes from other math logic.

Conversions to IEEE floating point types are implicit when no loss of precision is possible. The
effective bits of precision for this are given in the table below.

| Type       | Bits of Precision |
| ---------- | ----------------- |
| `float16`  | 11                |
| `float32`  | 24                |
| `float64`  | 53                |
| `float128` | 113               |
| `float256` | 237               |

### Implicit Optional conversions

For a reference type `R`, the reference type is a subtype of the optional type `R?`. However, that
is true only for the first layer of optional type. Given an optional reference type `R?` it is not
itself a reference type and thus `R?` is not a subtype of `R??`. For all types `T` other than
reference types, there is an implicit conversion from `T` to `T?`.

Note that a generic type parameter `T` is not known to be a reference type. So `T` to `T?` is an
implicit conversion unless the generic type parameter has been constrained to be a reference type.

### Implicit Lifted Conversions

Given two types `S` and `T` such that `S <: T` then `S? <: T?`. That is, optional types are
covariant to their underlying types. Put another way, the underlying type acts like an `out` generic
parameter. If no subtype relationship exists, but instead there is an implicit conversion from `S`
to `T`, then there is a lifted implicit conversion from `S?` to `T?`.

### User-defined Implicit Conversions

**TODO**

## Explicit Conversions

The following conversions are explicit conversions:

* Explicit numeric conversions
* Explicit boolean conversions
* Explicit reference conversions
* Explicit lifted conversions
* Explicit boxing conversions
* Explicit unboxing conversions
* User-defined explicit conversions

Explicit conversions may be failable or non-failable. Non-failable conversions can be performed with
the `as` operator. This operator additionally allows one to explicitly use implicit conversions.
Failable conversions can be performed with the "`as!`" or "`as?`" operators. The former causes an
abort when the conversion fails at runtime, the latter produces the value "`none`". Thus an
expression `x as? T` produces a value of the type `T?`.

### Explicit Numeric Conversions

The explicit numeric conversions are all the conversions between numeric types for which an implicit
conversion doesn't exist. An explicit numeric conversion fails if the value being converted is not
in the range of the target type.

Conversions from any integer numeric type, including "`size`" and "`offset`", to "`float32`" or
"`float64`" will not fail when the floating point type has fewer bits of precisions. However, they
may cause a loss of precision. These can be performed using the "`as`" operator.

### Explicit Boolean Conversions

Conversions from `bool` to any numeric type may be made explicitly with `false` converting to 0 and
`true` converting to 1. These conversions cannot fail, but require explicit conversion with the `as`
operator.

### Explicit Reference Conversions

Given reference types "`S`" and "`T`", there is an explicit conversion from "`S`" to "`T`" unless it
can be proven that there does not exist a type "`U`" where `U <: S` such that `U <: T`. A further
restriction is that if the type `S` is an `iso` or `own` drop type then the type `T` must also be an
`iso` or `own` drop type. This restriction ensures that the `move` nature of the type cannot be lost
so that the compiler can ensure the `move` value is deleted.

**TODO:** maybe this should require a `cast[T](v)` instead to be more explicit?

### Explicit Lifted Conversions

Given two types `S` and `T` for which there is an explicit conversion from `S` to `T`, there is a
lifted explicit conversion from `S?` to `T?`. The fallibility of this conversion matches that of the
underlying conversion.

### Explicit Boxing Conversions

Given a value or struct type `V` and a trait type `T` for which `V` implements `T`, there is an
non-failable explicit boxing conversion from `V` to `T`. A boxing conversion allocates space on the
heap and puts the value in that box. The boxed type implements all the traits that the value or
struct type implements. Value types are copied into the box. Struct types are moved into the box. A
drop type can be boxed with `move v as T`. This moves the value into the box. However, this is
supported only when the type `T` is a drop type.

### Explicit Unboxing Conversions

Given a value or struct type `V` and a trait type `T` for which `V` implements `T`, there is an
failable explicit unboxing conversion from `T` to `V`. For value types, this copies the value out of
the box. For struct and drop types, `move t as V` will move the value out of the box.

### User-defined Explicit Conversions

**TODO**
