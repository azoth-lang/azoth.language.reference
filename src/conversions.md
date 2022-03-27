# Conversions

A *conversion* is a transformation of values from one type to another that allows expressions of the first type to used as the second type. Conversions can be *implicit* or *explicit*. Explicit conversions require an explicit conversion expression. For example, the type "`int32`" can be implicitly converted to "`int64`", so expressions of type "`int32`" can implicitly be treated as "`int64`". The opposite conversion, from "`int64`" to "`int32`, is explicit, so an explicit conversion is required.

```azoth
let a: int32 = 123;
let b: int64 = a;       // implicit conversion from int32 to int64
let c: int32 = b as! int32; // explicit conversion from int64 to int32
```

Additionally, certain numeric conversions will not fail, but may lead to a loss of precision. These are described in [Explicit Numeric Conversions](#explicit-numeric-conversions) and can be performed using the "`as`" operator.

Some conversions are defined by the language. Programs may also define their own conversions (see [User-defined Conversions](#user-defined-conversions)).

## Implicit conversions

The following conversions are implicit conversions:

* Implicit numeric conversions
* Implicit optional conversions
* Implicit constant expression conversions
* User-defined implicit conversions

The "`as`" operator can be used to explicitly cause an implicit conversion.

### Implicit Numeric Conversions

The implicit numeric conversions are those conversions between simple numeric types that can be safely performed with no loss of precision.

The implicit numeric conversions are:

| From      | To                                                                                           |
| --------- | -------------------------------------------------------------------------------------------- |
| `int8`    | `int16`, `int32`, `int64`, `int`, `float32`, `float64`                                       |
| `byte`    | `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`, `float32`, `float64` |
| `int16`   | `int32`, `int64`, `int`, `float32`, `float64`                                                |
| `uint16`  | `int32`, `uint32`, `int64`, `uint64`, `int`, `uint`, `float32`, `float64`                    |
| `int32`   | `int64`, `int`, `float64`                                                                    |
| `uint32`  | `int64`, `int`, `uint`,`uint64`, `float64`                                                   |
| `int64`   | `int`                                                                                        |
| `uint64`  | `int`, `uint`                                                                                |
| `float32` | `float64`                                                                                    |

### Implicit Optional conversions

Given a value type `S` and reference type `T` such that `S <: T`, there are implicit boxing conversions from "`S?`" to "`T?`" and from "`S`" to "`T?`". There is also an implicit conversion from `S` to `S?`.

### Implicit Constant Expression Conversions

A constant expression of the types "`int8`", "`byte`", "`int16`", "`uint16`", "`int32`", "`uint32`", "`int64`", "`uint64`", "`int`", "`uint`", "`size`", "`offset`", "`float32`", or "`float64`" can be implicitly converted to any other type in the list if the value of the constant expression is within the range of the destination type and can be represented without loss of precision. Thus for conversion to "`float32`" and "`float64`" the value must not have more significant bits than be represented by the type.

### User-defined Implicit Conversions

**TODO**

## Explicit Conversions

The following conversions are explicit conversions:

* All implicit conversions
* Explicit numeric conversions
* Explicit reference conversions
* User-defined explicit conversions

Explicit conversions can be performed with the "`as!`" or "`as?`" operators. The former causes abandonment when the conversion fails at runtime, the latter produces the value "`none`".

### Explicit Numeric Conversions

The explicit numeric conversions are all the conversions between numeric types for which an implicit conversion doesn't exist. An explicit numeric conversion fails if the value being converted is not in the range of the target type.

Conversions from any integer numeric type including "`size`" and "`offset`" to "`float32`" or "`float`" will not fail but may cause a loss of precision. These can be performed using the "`as`" operator.

### Explicit Reference Conversions

Given reference types "`S`" and "`T`", there is an explicit conversion from "`S`" to "`T`" unless it can be proven that there does not exist a type "`U`" where `U <: S` such that `U <: T`.

### User-defined Explicit Conversions

**TODO**
