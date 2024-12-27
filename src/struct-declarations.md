# Struct Declarations

## Implementing Traits

Structs can implement traits. When doing so, the reference capabilities of the `self` parameter must
be mapped to proper self parameters on the struct. Even though `Self` is similar to a type
parameter, it has the same mapping despite the fact that it isn't the mapping used for other type
parameters. Here is the acceptable mapping.

| Trait        | Struct            |
| ------------ | ----------------- |
| `iso self`   | `move self`       |
| `mut self`   | `ref var self`    |
| `self`       | `ref self`        |
| `const self` | `ref const self`? |
| `id self`    | ??? `id ref self` |

What about `lent`?

The reason for this mapping to use `ref` is to make it possible for these methods to be called on a
stack based value. However, it might make sense to allow them to optionally be `iref`. Any methods
actually called via the interface will be called on boxed values for which `iref` is valid.

**NOTE**: `lent` is not a replacement for `ref` because a lent reference can be passed up the
stack past the original declaration in a way that `ref` can't. Also, `lent` can apply to a copy
struct that contains references to restrict how those references can be used.

**NOTE:** `ref const self` makes sense since you can declare constants like `const x: usize = 5`. If
you take a reference to that, it could just be `ref usize`. However, by symmetry with `ref var`, a
`ref const` makes sense and allows for the constant to be passed between threads and meet the
expectations of a trait using a `const self` parameter.

## Record Structs

Record structs provide a shorthand for declaring record types.  That is types that are the
composition of a named, ordered sequence of other types. They work just like record classes. A
record struct is declared with a parameter list after the struct name (e.g. `struct Example(foo:
int, bar: string)`). These parameters then become fields of the type. Declaring a record struct
implicitly declares three things in the struct. Additional members can be declared in additional to
the implicitly declared ones.

First, each of the parameters is declared as a field of the struct. For this reason, the parameters
may not be `lent`. A parameter may be declared `var` in which case the corresponding field will be
declared `var`. Of course a `const` struct cannot have `var` fields. Each of the implicitly declared
parameters is `public` unless the struct is `published` in which case they are declared `published`.
These fields can implicitly override abstract getters and setters. They cannot override or hide
non-abstract ones.

Second, an initializer is declared with corresponding field parameters (e.g. `init(.foo, .bar)` for
the `Example` struct). The implicitly declared initializer is `public` unless the struct is
`published` in which case it is declared `published`. This initializer is otherwise not special and
additional initializers may be declared. Those initializers are not required to call the record
initializer. They must simply follow the initialization safety rules of all initializers. It is not
possible to add logic to this initializer or call a base class initializer. However, if an
initializer with the same parameter types is declared, it replaces the initializer that would
otherwise be generated. That initializer may contain logic and call a base or other initializer.

Third, a pattern match deconstructor is declared corresponding the the parameters/fields (e.g.
`operator is(self) -> Tuple[int, string]` for the `Example` struct). This allows the record struct
to be deconstructed as part of an assignment or pattern match. If another deconstuctor is explicitly
declared, the implicit one is still generated. It is not possible to replace this deconstructor. If
control of the deconstructor is needed, then a regular struct should be declared and members that
would be implicitly declared by a record struct can be manually written.
