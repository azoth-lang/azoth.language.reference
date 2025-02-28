# Optional Types

Optional types can have all values of an underlying type plus an additional value "`none`". The
underlying type can be any type including value or reference types.

```grammar
optional_type
    : type "?"
    ;
```

## Subtyping and Conversions

For all reference types `T` and `U` if `T <: U` then `T <: T? <: U?`. However, given value type `V`
and reference type `R` such that `V <: R`, the type `V` is not a subtype of `V?` and `V?` is not a
subtype of `R?`. There is however an implicit conversion from `V` to `V?` and an explicit boxing
conversion from `V?` to `R?`.

If there is an implicit or explicit conversion from `V` to `W` then there is a corresponding lifted
conversion from `V?` to `W?`.

## The "`none`" Value

The special value "`none`" is used to represent when an optional type does not have a value. The
value "`none`" has the type "`never?`" thus it can be assigned into any optional type.

## Conditioning on a Value

To conditionally operate on an optional value, use a pattern match expression. Other ways of
checking for "`none`" are possible but not preferred.

```azoth
let x: int? = ...;

// Idiomatic way of checking for `none`
if x is let y?
{
    // y is the value of x
}

// Not Recommend unless part of a larger match
match x
{
    y? => ...,
    none => ...,
}

// Not Recommend
if x =/= none
{
    // Can't directly use the value of `x` in this block
}
```

## Coalescing Operators

The coalescing operator `??` allows for the replacement of `none` with another value. If the
expression to the left of the operator evaluates to a value other than `none` then the result of the
coalescing operator is that value and the right hand expression is not evaluated. Otherwise, the
result is the result of evaluating the right hand expression. Note that this is a short circuiting
evaluation. The coalescing operator can be combined with assignment as `??=` so that the left-hand
side is evaluated and if it is `none` then the right-hand side is evaluated and assigned into the
left-hand side.

**TODO:** how does this operator work on multiple levels of optional (i.e. `T??`)?

## Conditional Access

Members of optional values can be accessed using the conditional access operator `x?.y`. This
operator evaluates the left hand side. If the left hand side evaluates to `none`, then the right
hand side is not evaluated and the result of the expression is `none`. Otherwise, the member is
accessed and evaluated. Note that this is a short circuiting evaluation so that `x?.y.z()` would
prevent the method `z` from being called if `x` were `none`, rather than simply evaluating `x?.y` to
`none` and then attempting to call `z()` on it.

## Operator Lifting

For a type `T` with operators defined on it, many of those operators are lifted to operate on the
type `T?`. The operators that can be lifted fall into three categories based on how they are lifted.
The three categories are the arithmetic operators, comparison operators, and logical operators.

The arithmetic operators `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=` (both unary and binary) are
lifted so that if either argument is `none` the operator result is `none`.

The comparison operators `==`, `=/=`, `<`, `<=`, `>`, `>=` are lifted to that `none == none` but the
comparison of `none` to any other value results in `none`. Thus `none == none`, `none <= none`, and
`none >= none` are all true since `none` is equal to itself. Likewise `none =/= none`, `none <
none`, and `none > none` are all false. Comparison on optional types is a proper partial order where
`none` is comparable to itself but not to any other value.

The logical operators `not`, `and`, `or`, `xor`, `implies`, `iff` are lifted to operate on `bool?`
according to three-valued logic where the value `none` represents an unknown truth value. They
follow the Kleene logic. The truth tables are given below. The logical operators are still short
circuiting, however the `none` value can influence whether the second argument is evaluated. For
example, it is still the case that `true or x` will not evaluate `x`, but `none or x` must evaluate
`x` to determine whether the result is `none` or `true`.

| `a`     | `not a` |
| ------- | ------- |
| `false` | `true`  |
| `none`  | `none`  |
| `true`  | `false` |

(symmetric cases omitted)

| `a`     | `b`     | `a and b` | `a or b` | `a xor b` | `a iff b` |
| ------- | ------- | --------- | -------- | --------- | --------- |
| `false` | `false` | `false`   | `false`  | `false`   | `true`    |
| `false` | `none`  | `false`   | `none`   | `none`    | `none`    |
| `false` | `true`  | `false`   | `true`   | `true`    | `false`   |
| `none`  | `none`  | `none`    | `none`   | `none`    | `none`    |
| `none`  | `true`  | `none`    | `true`   | `none`    | `none`    |
| `true`  | `true`  | `true`    | `true`   | `false`   | `true`    |

| `a`     | `b`     | `a implies b` |
| ------- | ------- | ------------- |
| `false` | `false` | `true`        |
| `false` | `none`  | `true`        |
| `false` | `true`  | `true`        |
| `none`  | `false` | `none`        |
| `none`  | `none`  | `none`        |
| `none`  | `true`  | `true`        |
| `true`  | `false` | `false`       |
| `true`  | `none`  | `none`        |
| `true`  | `true`  | `true`        |

**TODO:** how does lifting work on multiple levels of optional (i.e. `T??`)?

## Conditioning on `bool?`

The `if` and `while` expressions both have a condition which is normally a `bool`. They can also
accept `bool?`. When conditioning on `bool?`, the value `true` is the only true value. This is what
distinguishes the `bool?` logic as Kleene logic instead of Priest logic. This a value of `none` will
cause the `else` to execute and a `while` to exit.

**TODO:** how does condition work on multiple levels of optional (i.e. `bool??`)? Should it simply
be ill-typed or should it be allowed with any level of `none` being not true?

## Optional Types Precedence

The optional type is immutable so `mut T?` must mean `(mut T)?`.

For value type `V` and reference type `R` the optional types have the following meanings.

| Type     | Meaning    |
| -------- | ---------- |
| `V?`     | `V?`       |
| `mut V?` | `(mut V)?` |
| `R?`     | `R?`       |
| `mut R?` | `(mut R)?` |

## Optional Type Implementation

*This section is meant only as guidance how optional types might be implemented. The actual
implementation may be anything that implements all language requirements.*

For an optional reference type, the bit representation is the same as a reference. The `none` value
is represented by a null reference. That is, all the bits of the reference are zero.

For all other types `T`, optional types act similarly to a constant value type `Option[T out]`.
Multiple layers of optional (e.g. `T??`) can be thought of as nesting (e.g. `Option[Option[T]]`).
However, the recommended representation is able to represent both a single layer of optional and up
to 255 layers of optional. Internally, they may be represented as the bits of the type `T` followed
by a single `byte`. If the value is not `none` then the bits of the value come first followed by
zero for the byte. If the value is `none` then the value bits should be zero and the byte contains
an integer indicating at which level the `none` occurs. That is a `none` for `T?` would be `1`.
However, for type `T??` there are two `none` values. If the inner `T?` is `none` then the byte has
the value `1`. If the outer optional of `(T?)?` is `none` then the byte value is `2`. Thus this
representation can represent up to 255 layers of nesting since the maximum value of `byte` is 255.
If it is necessary to represent more than 255 layers of optional, then this representation can be
nested. Thus if this representation is like a value type `Multi_Option[T]` then more than 255 layers
can be represented by `Multi_Option[Multi_Option[T]]`.

It should be noted that for a reference type `R`, the type `R?` is not a reference type. It would
need the above representation. Thus for the type `R??`, the inner optional being `none` is
represented by a zero reference, while the outer optional being `none` is represented by the value
`1` in the `byte` following the reference bits.

The reasons this is a good representation are as follows:

* Reference types are optimized for the simple case and have the same representation reflecting the
  fact that `R <: R?` for reference types.
* Implicit conversion to optional requires only appending a zero byte.
* Implicit conversion to additional layers of optional typically is a no-op.
* A single byte provides more than enough layers of optional for typical cases, but can also be fit
  within the space of many value types that contain values smaller than the native word size (i.e.
  if a value type contains a `bool` or `byte` then it will already have a representation that isn't
  an even multiple of the native word size. Thus it will need to be padded out for alignment. The
  optional version of this type can then place the additional byte within that padding so that the
  aligned size of the optional type is no bigger.)
