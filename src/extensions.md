# Extensions

## Type Extensions

A type (i.e. class, trait, or struct) can be extended with additional traits and methods using an
extension. Extensions modify the original type and as such the namespace they are declared in
doesn't matter except that the extension must be visible to the consuming code. Extensions modify
types. That is why the `class`, `trait`, or `struct` keyword is omitted. As such they can do things
that aren't possible with regular declarations.

```azoth
public extend Class <: Trait
{
}
```

To publish an extension outside of a package, one of the types involved must be declared in that
package, or the package must be trusted. For example, a packaged can publish an extension to
existing types to make them implement a new trait if it declares that trait. However, to publish
additional methods on the type that use only existing types, it must be trusted. This rule helps to
prevent breaking changes and ambiguities. Note that a package can internally declare and use any
extension it wants.

As stated before, extensions can be applied to types. This allows an extension to be applied to
specific reference capabilities etc.

**TODO:** how does this work with the fact that read only is the default?

```azoth
// All constant references can be shared between threads.
public extend const Any <: Sendable { }

// All references are nonzero
public extend[T] T <: Nonzero
    where T: reference { }
```

Extensions participate in the virtual method dispatch of the type. They can be overridden in
subtypes and can override base methods.

Note: extensions have access to protected members of their class

## Extension Members

Extension members act like extension methods in C# they are statically dispatched. They are declared
outside of any type. They are in scope only if the namespace they are declared in is imported. (Note
that by default all namespaces are searched. However, `using` statements can modify that.) Extension
methods are distinguished by being declared with a first parameter of `self`.

```azoth
public fn example(self: int, x:int) -> int => self + 2 * x;
```

**TODO:** should their be a different way to call extension methods (e.g. `obj..method()`)?

**TODO:** are extension members needed in the language given that type extension is available?

## Notes

Swift protocols mix required methods which "dispatch dynamically" with extensions which "dispatch
statically". That seems really confusing. This is connected to the idea in Rust that you don't
arbitrarily extend structs, but rather, you implement traits for them. That provides structure to
the idea of dispatch. You are adding that functionality only when you can see it has having that
type.

Another way to think of this is the difference between interface methods in C# and extension
methods. That makes it seem like the two should be clearly different syntaxes. Luckily, we have a
ready made syntax for this. Consider the following function outside any class.

```azoth
public fn my_method(self: Example) -> int
{
    return self.field * 2;
}
```

That clearly looks like an extension that is outside of the class and would be statically
dispatched. It could be invoked using regular function syntax by qualifying it with the namespace.
This even allows extension properties and operators! (Though operators should perhaps just be static
functions instead.)

```azoth
public get property(self: Example) -> int
{
    return self.field * 2;
}

public operator +(self: int, x: Example) -> int
{
    // ...
}
```
