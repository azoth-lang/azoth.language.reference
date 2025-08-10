# Pointer Types

Pointer types are used by unsafe code to directly address memory. Only pointers to unmanaged value
types are supported. This ensures that pointers can be implemented as simple addresses compatible
with pointers in languages like C. Reference types may require fat pointers to enable virtual method
dispatch. Thus pointers to reference types are not valid.

Pointer types are non-nullable. To represent a nullable pointer use an optional pointer type (i.e.
`@int32?`).

```grammar
pointer_type
    : "@" "var"? value_type
    | "@" "var"? "void"
    ;
```

**TODO:** function pointers `@(int32, int32) -> void`

## Unmanaged Types

Pointers must point to unmanaged types. The following are unmanaged types:

* Simple types except `int` and `uint`
* Pointers
* Structs composed only of unmanaged types (except `closed` structs)

This restriction ensures that pointers are constructed only to types which do not contain any
garbage collected references. The simple types `int` and `uint` are not unmanaged types because even
though they may consist of a locally stored value, when the value they contain grows large, they
often contain a garbage collected reference to the larger number.

Closed structs are not unmanaged types because they contain a discriminator that does not have a
guaranteed memory layout that can be handled from C/C++ code.

Note that structs should generally by external or attributed with layout sequential, or the memory
layout will be unknown an may change between versions of the runtime or in unexpected ways between
platforms.

## Void Pointer Type

The type "`@void`", is a pointer type that places no limits on what may be at the memory pointed to.
Conversion of a pointer to the void pointer type can be done implicitly. Conversion from a void
pointer to another pointer type requires an explicit cast.

## Mutability

There are two types of pointers. Read-only pointers `@T` do not allow any mutation of the value
through the pointer, but do not guarantee that nothing else can modify the value. Mutable or
variable pointers `@var T` allow assignment and mutation through the pointer. If the type `T` is
*not* `const`, then `@var T` indicates that the memory may be assigned into (i.e. `var`) *and* that
the value there may be mutated (i.e. `mut`).

## Creating Pointers

There are several ways to create pointers. Note that creating a pointer is not unsafe. It is only
dereferencing a pointer that is unsafe.

### Address of a Local or Struct Field

A pointer to a local binding can be gotten using the address of operator '`@`' (e.g. `@x`). This
operator produces a variable pointer (i.e. `@var T`) if it is taking the address of a binding
declared with `var` or a `mut` type. Note that this does mean that in some instances it will expand
what can be done to the value. The address of operator can also be used to take the address of a
field of a struct stored in a local binding. For example, given `var s: Some_Struct = ...`, the
expression `@s.f` will produce a pointer to the field `f` in the value `s`.

### Fixed Statement

To get the address of a field in an object on the heap or to convert a `Var[T]` or `Ref[T]` to an
address a `fixed` statement must be used to pin the object location while the pointer exists.
Without this, the garbage collector could attempt to move the object thereby invalidating the
pointer.

To take the address of an object field, simple use the `@` operator on a field of `self` inside the
fixed statement:

```azoth
fixed let ptr = @.f
{
    // use ptr here
}
```

Note that this only works on `self` because accessing fields on other objects may happen through a
property getter even if it is declared with field syntax.

To cast a `Var[T]` or `Ref[T]` to a pointer, do the conversion with `as` inside of a fixed statement.
Among other things, this allows for pointers to list and array members.

```azoth
fixed let ptr = list.at(43) as @var int32
{
    // use ptr here
}
```

## Dereferencing

Pointers can be dereferenced with the prefix unary dereference operator '`^`' (e.g. `^x`). To access
members the dereference member operator '`^.`' can be used. Dereferencing a pointer is an unsafe
operation that must happen in an unsafe context.

```azoth
var x: Example = Example(1);
^x = Example(42);
let y = x^.value;
```

## Calling Methods

Methods can be called on pointers using the `^.` operator. When doing so, the pointer is converted
to the appropriate type for the self parameter. If the self parameter is passed by value then the
value pointed to is copied and passed as `self`.
