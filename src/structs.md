# Structs

Structs are similar to classes. They contain field and function members. Unlike classes, structs are
value types and by default are allocated on the stack instead of the heap. A variable with a struct
type directly contains the data of the struct. A variable of a class type contains a reference to
the object which contains the data.

```grammar
struct
    : access_modifier "mut"? "move"? "ref"? "struct" identifier //...
    ;
```

## Struct Kinds

There are two kinds of structs. This kind determines the semantics of assigning values of that type.
By default structs are copied. However, you can declare a struct with `move` similar to `move`
classes. This causes the struct to be moved instead of copied. This leaves behind an `id` struct.

| Struct Type  | Declaration Syntax | Semantics                                                              |
| ------------ | ------------------ | ---------------------------------------------------------------------- |
| Move Structs | `move struct`      | Struct values "move" when assigned so that they can no longer be used. |
| Copy Structs | `struct`           | Struct values are implicitly copied when assigned.                     |

The kind of struct determines the default assignment behavior, but it is often possible to
explicitly cause the other behavior.

These kinds are independent of whether the struct is a `ref struct` (see [Ref
Structs](ref-structs.md)).

## Mutable Structs

Structs are implicitly declared as `const` types the way `const class` works. To have a non-const
struct declare it `mut struct`.

## Pseudo-Reference Structs

**TODO:** Redo section now that structs have reference capabilities

Sometimes it is useful to create a struct that acts as if it were a simple reference to an object on
the heap down to even having a reference capability. For example, this is how array slices are
implemented efficiently. To allow this, the struct must be distinguished from a regular struct which
would not be prefixed by a reference capability, and the reference capability of a given declaration
must be available to the code of the struct. These structs are declared with the `ref[`*C*`]`
syntax. This should not be confused with a standard `ref` struct which allows stack reference types
to be stored in the struct. Instead, a pseudo-reference struct causes the compiler to treat the
struct type as if it were a reference type and give each declaration of the type a reference
capability that is made available within the the struct as the capability *C*.

Pseudo-reference structs may have all the other standard struct modifiers. Most pseudo-references
should be `copy` structs, but a `move` struct allows for the pseudo-reference to act as if it were a
`move class` and have a destructor or reference a `move class`. A `ref` pseudo-reference confines
the pseudo-reference to the stack and allows it to contain stack references.
