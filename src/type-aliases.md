# Type Aliases

**TODO:** If function and method aliases won't be added in the future, then these should perhaps
just use the `type` keyword without an `alias` keyword.

Type aliases allow types to be given new names and optionally to expose those names from new
namespaces. A type alias is declared with `type alias` followed by the alias name and any generic
parameters. This is assigned to another type. Any constraints on the type parameters come last. It
does not create a separate type only a synonym for an existing type.

```azoth
public type alias Promise_Result[T] = Promise[Result[T]]
    where T: Example;
```

Private aliases are useful for creating local shorthands for long types. However, an alias can be
made `public` or even `published` to allow it to be used from other places.

When a type alias occurs inside of type declaration, it is an associated member. That is, it is
accessed from the type not from any instances.
