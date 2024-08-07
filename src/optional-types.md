# Optional Types

Optional types can have all values of an underlying type plus an additional value "`none`". The
underlying type can be any type including value or reference types.

```grammar
optional_type
    : type "?"
    ;
```

## Subtyping

For all reference types `T` and `U` if `T <: U` then `T <: T? <: U?`. However, given value type `S`
and reference type `T`, the type `S` is not a subtype of `T?`. However, there is an implicit
conversion.

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

// Not Recommend
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

## Coalescing Operator

The coalescing operator `??` allows for the replacement of `none` with another value. If the
expression to the left of the operator evaluates to a value other than `none` then the result of the
coalescing operator is that value and the right hand expression is not evaluated. Otherwise, the
result is the result of evaluating the right hand expression. Note that this is a short circuiting
evaluation.

## Conditional Access

Members of optional values can be accessed using the conditional access operator `x?.y`. This
operator evaluates the left hand side. If the left hand side evaluates to `none`, then the right
hand side is not evaluated and the result of the expression is `none`. Otherwise, the member is
accessed and evaluated. Note that this is a short circuiting evaluation so that `x?.y.z()` would
prevent the method `z` from being called if `x` were `none`, rather than simply evaluating `x?.y` to
`none` and then attempting to call `z()` on it.

## Operator Lifting

Operators are lifted for optional types similar to how they are in C#.

## Optional Types Precedence

The optional type is immutable so `mut T?` must mean `(mut T)?`.

For value type `V` and reference type `R` the optional types have the following meanings.

| Type             | Meaning            |
| ---------------- | ------------------ |
| `V?`             | `V?`               |
| `mut V?`         | `(mut V)?`         |
| `R?`             | `R?`               |
| `mut R?`         | `(mut R)?`         |
