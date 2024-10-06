# Struct Initializers

Structs are allocated on the stack instead of the heap. To reflect this, structs are not constructed
using the `new` keyword. Instead, they are initialized using initializer functions. Initializer
functions are similar to constructors in that they enforce definite assignment. However, they are
called more like associated functions.

An initializer may be named or unnamed. They are declared using the `init` keyword.

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

Like constructors, initializers have a self parameter. However, this parameter is passed `ref mut
self` so no copying or initialization is necessary to invoke the initializer.

## Default Initializers

A struct without any initializers will have a default initializer generated for it.

## Field Initialization Shorthand

Just like constructors, there is a shorthand for initializing fields.

```azoth
public init(ref mut self, .field)
{
}
```

## Definite Assignment

Like constructors, definite assignment of fields is enforced. Since a struct never has a base class,
the transition point to fully initialized is always implicit.

## Copy Initializers

**TODO:** document how copy works on struct types. Or should struct copy be required to be bitwise?
