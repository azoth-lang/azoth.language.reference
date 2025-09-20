# Categories of Types

In Azoth there are broadly three categories of types. There are reference types, value types, and
hybrid types. For each, there is a corresponding way of declaring them. Other types are constructed
from the types in these categories and themselves fall into one of the three categories.

## Reference Types

Reference types have values that are allocated dynamically on the heap. The garbage collector is
responsible for reclaiming memory for reference type values when the value becomes unreachable.
Reference typed variables, parameters, fields, etc. do not actually contain the value. Instead they
are always a reference to the value on the heap. Reference types provide the maximum flexibility but
the indirection of the reference can impact performance. This performance cost is most problematic
for small constant values and for high-performance code.

Reference types allow the full flexibility of capabilities to apply. In particular, for a reference
type `T`, the type `id T` is a reference which can always be freely shared between threads, does not
prevent recovery of isolation or freezing, and which can cross the boundary of isolated object
subgraphs.

New reference types are declared primarily with class and object declarations. A class declaration
is a sort of template for instances of the class. While an object declaration declares a unique
instance for the type. Each trait declaration also creates a reference type. See [Traits](#traits)
for the nuances of traits with regards to reference, value, and struct types.

### Other Reference Types

In addition to reference types from class and trait declarations, the following types are reference
types:

* Function Types
* Intersections of Reference Types
* Unions of Reference Types
* The special types `Any`, `Type`, and `Metatype`

## Value Types

Value types have values that are allocated directly on the stack, in object fields, etc. This is
often shortened to simply saying that are stack allocated. However that is not technically correct
since a value type in an object field is allocated on the heap as part of an object instance.
Additionally, there are times when a value type that would normally be on the stack must be moved to
the heap. For example, local variables can be moved to the heap when they are captured as part of a
closure. It is perhaps most accurate to say value types are allocated inline since for any value
typed variables, parameters, fields, etc. the value is stored directly. Value types are passed by
value meaning that when passing the value to a function or assigning a value from one variable to
another, the value is copied so that there are now two copies of the value, one inline in each
place.

Value types also allow the full flexibility of capabilities to apply. While they are not themselves
references, they can contain references to which the capabilities apply.  The capabilities on value
types act exactly as if the value was a reference. For example, an `iso Value` will become an `own
Value` or `mut Value` when a copy is made the same way an isolated reference would when a copy of
the reference was made.

New value types are declared with value declarations. A value declaration is a sort of template for
constructing values of a given type. Since values are copied, they limit their fields to be `let`
bindings only (`var` bindings are not allowed). Additionally, they cannot contain fields that are
isolated or owned since those could not be properly copied without introducing the risk of
divergence between the copies.

Value types can be drop types. In that case, any `own` value that goes out of scope will have the
drop method called on it. This allows values to contain references to drop types.

### Other Value Types

In addition to value types declared with value declarations, there are a number of other value
types. Many of the built-in types are value types as are several constructed types. The following
types are value types:

* `int`, `uint`
* The fixed size and platform sized integer types (e.g. `int32`, `uint32`, `size`, `offset`, `nint`,
  `nuint`)
* `bool`
* `float16`, `float32`, `float64`
* Unions of value types
* Optional types (i.e. `T?`)
* Tuple types
* Pointer types (i.e. `@T`)

Optional types and tuple types are themselves value types, however they can contain reference types.
In that way, they can act similar to reference types.

## Hybrid Types

Many languages have reference types and value types (e.g. C#, Java). Hybrid types are unique to
Azoth since they are enabled by capabilities and most languages do not have capabilities. A hybrid
type is allocated inline like a value type, but passed by reference like a reference type. This
split behavior is determined by the capability. For a hybrid type `H`, the isolated type `iso H` is
stored inline in variables, parameters, fields, or even in a temporary. To relocate this value it
must be moved to another location. Using a `move` ensures that the resulting value is again
isolated. Any other capability (e.g. `mut H`, `H`) is stored and passed as a reference to where the
value is stored. Thus these references can reference a value that might be on the heap or might be
in a field as part of an object. This means that a hybrid type reference references places that
cannot be referenced by a reference type. A reference to a value on the stack is sometimes referred
to as a variable reference. An example of a language that supports this is C# using its `ref`, `in`,
and `out` keywords. A reference to a field within a value on the heap is called an interior
reference. An example of a language that supports these in a limited way is Go. In Go, an array
slice is an interior reference into the middle of an array.

Due to their unique nature, hybrid types impose several limitations. Hybrid types support the
special capability `own`. When an `iso` hybrid type value is aliased, it will become `own`. The
compiler will ensure that owned hybrid type values hav isolation recovered before the value goes out
of scope. This ensures that no hybrid type reference can outlive the value it references on the
stack since that reference would prevent recovery of isolation. To ensure the safety of references
to the stack, for a hybrid type `H`, the type `id H` breaks isolation and causes sharing between
object graphs. That is, for a hybrid type it is not possible to `move` it if there is an `id`
reference to it. For that reason it is also not possible to drop it when there is an `id` reference
to it. It might be thought that `const` could simply be disallowed for hybrid types. However, since
`const` is transitive and a hybrid type could be a field in an object, it is always possible that
the object becomes `const` and the field is therefore `const`.

Like reference types, hybrid types support comparison for reference equality (i.e. `@==`) and the
`identity_hash()` method. However, if a hybrid type value is moved, then the newly moved value will
have a different identity hash.

When a hybrid type is moved, the pervious variable binding type changes to `void` and reports errors
for "use of moved value".

New hybrid types are declared with struct declarations. All struct declarations are `move struct`
since all hybrid types are move types. A struct declaration is a sort of template for constructing
instances of the struct. There are not other hybrid types than those created with struct declarations.

### Uses of Hybrid Types

Hybrid types can be used to safely work with mutable data stored inline on the stack or in fields,
arrays, etc. This enables high-performance code with safety. However, they can also be used to
enable generalized stack and interior references.

The `Var[T]` and `Ref[T]` types allow one to declare a variable or field in a way that allows
references directly to it to be shared. A `Var[T]` internally contains a `var value: T` field that
can then be set through any references to the `Var[T]` instance. A `Ref[T]` internally contains a
`let value: T` field. This allows references to large value types to be passed without copying those
values. These provide the equivalent of C# `ref` and `in` keywords. There is no way to safely
emulate the functionality of the C# `out` keyword. However, it is far less necessary since tuples
and optional types can easily be returned from functions.

## Traits

Traits can be implemented by reference types, value types, and hybrid types. Indeed the same trait
can be implemented by all three. For example `Equatable` is commonly implemented by types in each
category. When used as generic constraints, traits do not determine the category of types that can
be passed to the generic parameter. Instead, various generic constraints can be used to do that.
However, when a trait is used as a type for a variable, parameter, field, etc. then it is a
reference type. Only reference type values that implement the trait can be passed. This is because
all references are the same size and carry information allowing them to be used polymorphically.
Value types and hybrid types vary in size and do not carry the information necessary for
polymorphism. Likewise, an instance of a value type or hybrid type that implements a trait cannot be
assigned into a variable of that trait. In languages like C# and Java assigning value types into
reference variables is allowed by an implicit boxing conversion. Azoth does not do implicit boxing.
This avoids the hidden performance costs of boxing which developers are often unaware of.

## Other Types

While almost all types fall into one of reference types, value types, and hybrid types, there are
two types that do not fit into these categories. The `void` type is in some ways an empty value
type. However, since it can make parameters drop out and doesn't allow `void` type variables,
parameters, fields etc. it isn't a true value type. The `never` type is a true subtype of types in
all three categories. In that regard in can be seen as being in all three categories. However, its
unique behavior and meaning puts it in a category of its own.
