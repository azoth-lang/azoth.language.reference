# Simple Types

The simple types are value types predefined by the Azoth language. They are identified by keywords.
To declare an identifier with the same name as one of these, use an [escaped
name](identifiers.md#escaped-identifiers). These keywords are fully distinct from the escaped name
with the same letters. So the "`bool`" type and a "`\bool`" identifier are fully distinct and can
never refer to each other.

```grammar
simple_type
    : numeric_type
    | "bool"
    ;

numeric_type
    : integer_type
    | floating_point_type
    ;
```

## Integer Types

The integer types provide signed and unsigned integers of various sizes.

```grammar
integer_type
    : "int"
    | "uint"
    | "int8"
    | "byte" // unsigned 8-bit
    | "int16"
    | "uint16"
    | "int32"
    | "uint16"
    | "int64"
    | "uint64"
    | "size"
    | "offset"
    | "nint"
    | "nuint"
    ;
```

### "`int`" and "`uint`" Types

The "`int`" and "`uint`" types are arbitrary size integers. These are sometimes referred to as "big
integers". They are capable of representing any integer subject to the memory storage available in
the computer. Generally, these are represented as regular integers up to some size a few bits less
than the native pointer size of the machine, and then as a reference to an array on the heap for
integers requiring larger bit sizes.

### "`nint`" and "`nuint`" Types

* `nint`
* `nuint`

The bit sizes of these are system dependent. Each has the native word bit length N of the system.
Typically this will be 16-bit, 32-bit, or 64-bit. It is assumed that the native bit length is at
least 16 bits. The `nint` has the range -2^(N-1) to 2^(N-1)-1 inclusive. The `nuint` has the range 0
to 2^N-1 inclusive.

### "`size`" and "`offset`" Types

* `size`
* `offset`

The numeric ranges of these are system dependent. "`size`" is an unsigned number large enough to
hold the maximum size of an array in Azoth. The `size` type is used to index into collections.
"`offset`" is a signed number, with the same positive and negative range as positive range of
"`size`". It is used to represent differences between indexes and pointers. Generally the size type
on an N-bit machine has the range 0 to 2^(N-1)+1 inclusive. The offset type then has the range
-2^(N-1) to 2^(N-1)-1 inclusive. Note that this means that the `offset` type has the same range as
`nint`, but the `size` type has a range constrained to a subset of `nuint` that doesn't correspond
to a simple reduction in bits in a two's complement representation.

The `size` and `offset` types are designed to reflect that fact that a negative index is invalid but
that an offset ought to be able to represent the different between the first and last index.

**TODO:** Different rules for arithmetic on them. For example `size + offset` ought to yield a
`size`.

## Floating Point Types

The types "`float16`", "`float32`" and "`float64`" are floating-point types represented as IEEE 754
binary16, binary32 and binary64 values.

```grammar
floating_point_type
    : "float16"
    | "float32"
    | "float64"
    ;
```

Floating point operations never cause an abort. In exceptional situations, they produce zero,
positive or negative infinity or not a number (NaN). Floating point operations may be performed with
higher precision than the result type of the operation. Some hardware architectures support higher
precision operations and implicitly perform all floating-point operations using this higher
precision type. Only with a performance impact can they be made to perform floating point operations
with lower precision. This rarely has an impact on the results of a numeric calculation. However in
expressions like "`x * y / z`" where the multiplication produces results outside the floating point
range and the division brings the temporary result back in range, the higher precision may cause a
finite result to be produced rather than infinity.

## "`bool`" Type

The "`bool`" type represents boolean values. The possible values of a boolean are "`true`" and
"`false`". The boolean logical operators "`and`" and "`or`" operate on boolean types.

No standard conversions exist between "`bool`" and other types. The bool type is distinct and
separate from the integer types. A bool value cannot be used in place of an integer value, and vice
versa.
