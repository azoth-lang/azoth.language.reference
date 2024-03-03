# Structs

Structs are similar to classes. They contain field and function members. Unlike classes, structs are
value types and by default are allocated on the stack instead of the heap. A variable with a struct
type directly contains the data of the struct. A variable of a class type contains a reference to
the object which contains the data.

```grammar
struct
    : access_modifier struct_capability "ref"? struct_kind pseudo_reference "struct" identifier //...
    ;

struct_capability
    : "const"
    | // epsilon
    ;

struct_kind
    : "move"
    | "copy"
    ;

pseudo_reference
    : "ref" "[" identifier "]"
```

## Struct Kinds

There are two main kinds of structs. This kind determines the semantics of assigning values of that
type. Additionally, there are enum structs documented in a separate section.

| Struct Type  | Declaration Syntax | Semantics                                                              |
| ------------ | ------------------ | ---------------------------------------------------------------------- |
| Move Structs | `move struct`      | Struct values "move" when assigned so that they can no longer be used. |
| Copy Structs | `copy struct`      | Struct values are implicitly copied when assigned.                     |

The kind of struct determines the default assignment behavior, but it is often possible to
explicitly cause the other behavior.

These kinds are independent of whether the struct is a `ref struct` (see [Ref
Structs](ref-structs.md)).

## Const Structs

Similar to classes, structs can be declared with the `const` modifier. This indicates that all
references inside of it must be `const` and that any field bindings are immutable (i.e. `let`). The
compiler can then assume that values of this type will not break the isolation of a reference.

## Pseudo-Reference Structs

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
