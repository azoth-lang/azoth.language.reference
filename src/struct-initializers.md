# Struct Initializers

Structs have initializers similar to classes, however there is no base initializer to call. The
initializer still has two phases and enforces definite assignment of fields before the `self` value
can be used.

```azoth
public drop struct Example
{
    public init(mut self)
    {
    }

    public init named(mut self)
    {
    }
}

let x = Example();
let y = Example.named();
```

## Default Initializers

A struct without any initializers will have a default initializer generated for it.

## Field Initialization Shorthand

Just like class initializers, there is a shorthand for initializing fields.

```azoth
public init(mut self, .field)
{
}
```

## Definite Assignment

Like class initializers, definite assignment of fields is enforced. Since a struct never has a base
class, the transition point to fully initialized is always implicit.
