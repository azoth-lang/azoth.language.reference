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
stack past the original declaration in a way that `ref` can't.

**NOTE:** `ref const self` makes sense since you can declare constants like `const x: usize = 5`. If
you take a reference to that, it could just be `ref usize`. However, by symmetry with `ref var`, a
`ref const` makes sense and allows for the constant to be passed between threads and meet the
expectations of a trait using a `const self` parameter.
