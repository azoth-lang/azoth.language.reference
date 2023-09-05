# `azoth.math` Namespace

## `decimal`

The math package defines the `decimal` type which is an arbitrary precision decimal number. This is
effectively stored as an `int` mantissa and a `uint` exponent with base 10. The basic operators and
operations like negation, addition, and subtraction are supported. The multiplication and division
operators require specifying a precision. It is likely that Azoth will support a `with` syntax that
among other things allows operators to be defined on the context. This would provide a convenient
way to specify precision when performing operations.

`decimal` should be the default inferred type for floating point constants.

```azoth
with decimal.precision(50) // i.e. 50 decimal digits
{
    let x = 1.0;
    let y = 3.0;
    let ratio = x/y;
}
```
