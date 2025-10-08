# Extensions

## Type Extensions

A type (i.e. class, trait, value, or struct) can be extended with additional traits and methods
using an extension. Extensions modify the original type and as such the namespace they are declared
in doesn't matter except that the extension must be visible to the consuming code. Extensions modify
types. That is why the `class`, `trait`, `value`, or `struct` keyword is omitted. As such they can
do things that aren't possible with regular declarations.

```azoth
public extend Class: Trait
{
}
```

To publish an extension outside of a package, one of the types involved must be declared in that
package, or the package must be trusted. For example, a package can publish an extension to
existing types to make them implement a new trait if it declares that trait. However, to publish
additional methods on the type that use only existing types, it must be trusted. This rule helps to
prevent breaking changes and ambiguities. Note that a package can internally declare and use any
extension it wants.

As stated before, extensions can be applied to types. This allows an extension to be applied to
specific reference capabilities etc.

```azoth
// All constant references can be shared between threads.
public extend const Any: Sendable { }

// All references are nonzero
public extend[T] T: Nonzero
    where T: reference { }
```

If no capability is specified then the bare type is being extended. To extend the read capability of
a type use the `read` keyword.

Extensions participate in the virtual method dispatch of the type. They can be overridden in
subtypes and can override base methods.

Note: extensions have access to protected members of their class
