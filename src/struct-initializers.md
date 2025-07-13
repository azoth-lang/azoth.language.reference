# Struct Initializers

Structs have initializers similar to classes, however there is no base initializer to call. The
initializer still has two phases and enforces definite assignment of fields before the `self` value
can be used.

Like with class initializers, they have a self parameter. However, this parameter is passed `ref mut
self` so no copying or initialization is necessary to invoke the initializer.

```azoth
public move struct Example
{
    public init(ref mut self)
    {
    }

    public init named(ref mut self)
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
public init(ref mut self, .field)
{
}
```

## Definite Assignment

Like class initializers, definite assignment of fields is enforced. Since a struct never has a base
class, the transition point to fully initialized is always implicit.

## Copy Initializers

Copy structs must be copied to be passed around. To do this, they use a special copy initializer. A
default copy initializer is generated that invokes the copy initializer on all fields. Ideally, this
would simplify to a bitwise copy of the whole struct. However, sometimes it is necessary to
customize the copy logic. This can be done by declaring a copy initializer.

A copy initializer is declared with the special name `copy` and takes aa argument of type`ref Self`
or `ref mut Self` which can optionally be `lent`.

```azoth
public init copy(ref mut self, value: ref Self)
{
    // copy fields
}
```

For any field that isn't explicitly assigned in the copy initializer automatically adds a copy
operation to the copy initializer.

Copy initializers cannot be explicitly called. They are implicitly invoked when a copy occurs.
