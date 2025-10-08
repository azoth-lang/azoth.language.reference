# Value Initializers

Values have initializers similar to classes, however there is no base initializer to call. The
initializer still has two phases and enforces definite assignment of fields before the `self` value
can be used.

Like with class initializers, they have a self parameter. However, this parameter is passed by value
and then the `self` value is returned by value.

```azoth
public const value example
{
    public init(self)
    {
    }

    public init named(self)
    {
    }
}

let x = example();
let y = example.named();
```

## Default Initializers

A value without any initializers will have a default initializer generated for it.

## Field Initialization Shorthand

Just like class initializers, there is a shorthand for initializing fields.

```azoth
public init(self, .field)
{
}
```

## Definite Assignment

Like class initializers, definite assignment of fields is enforced. Since a value never has a base
class, the transition point to fully initialized is always implicit.
