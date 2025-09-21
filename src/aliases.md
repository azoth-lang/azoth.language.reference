# Aliases

Aliases allow types and functions to be given new names and optionally to expose those names from
new namespaces.

## Type Aliases

A type alias is declared with `type alias` followed by the alias name and any type parameters. This
is assigned to another type. Any constraints on the type parameters come last. It does not create a
separate type only a synonym for an existing type.

```azoth
public type alias Promise_Result[T] = Promise[Result[T]]
    where T <: Example;
```

Private aliases are useful for creating local shorthands for long types. However, an alias can be
made `public` or even `published` to allow it to be used from other places.

When a type alias occurs inside of type declaration, it is an associated member. That is, it is
accessed from the type not from any instances.

## Function Aliases

**TODO:** is this worth the complexity of adding?

A function alias creates an alias for a function. When declaring a function alias it is not possible
to change the parameter types. Thus they are not listed in the alias declaration. However, they are
listed when stating the function being aliased in order to disambiguate overloads. An alias does
however let one modify generic parameters. Function aliases can be overloaded.

```azoth
public fn alias example[T] = example[T, T](T, T)
    where T <: Example;
```

## Method Aliases

**TODO:** should method aliases be supported as a symmetry with function aliases.
