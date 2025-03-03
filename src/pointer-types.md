# Pointer Types

Pointer types are used by unsafe code to directly address memory. Only pointers to unmanaged value
types are supported. This ensures that pointers can be implemented as simple addresses compatible
with pointers in languages like C. Reference types may require fat pointers to enable virtual method
dispatch. Thus pointers to reference types are not valid.

Pointer types are non-nullable. To represent a nullable pointer use an optional pointer type (i.e.
`@int?`).

```grammar
pointer_type
    : "@" value_type
    | "@" "void"
    ;
```

**TODO:** how do capabilities interact with pointer types? \
**TODO:** how does calling methods on pointer types work?

## Unmanaged Types

The unmanaged types are any of the simple types except `int` and `uint` and structs composed only of
unmanaged types. This ensures that pointers are constructed only to types which do not contain any
garbage collected references. The simple types `int` and `uint` are not unmanaged types because even
though they may consist of a locally stored value, when the value they contain grows large, they
often contain a garbage collected reference to the larger number.

## Void Pointer Type

The type "`@void`", is a pointer type that places no limits on what may be at the memory pointed to.
Conversion of a pointer to the void pointer type can be done implicitly. Conversion from a void
pointer to another pointer type requires an explicit cast.
