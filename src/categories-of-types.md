# Categories of Types

In Azoth there are broadly three categories of types. There are object types, value types, and
struct or *hybrid* types. For each, there is a corresponding way of declaring them. Other types are
constructed from the types in these categories and themselves fall into one of the three categories.

## Object Types

Object types have instances that are allocated dynamically on the heap. The garbage collector is
responsible for reclaiming memory for object type instances when the instance becomes unreachable.
Object typed variables, parameters, fields, etc. do not actually contain the instance. Instead they
are always a reference to the instance on the heap. Object types provide the maximum flexibility but
the indirection of the reference can impact performance. This performance cost is most problematic
for small constant values and for high-performance code.

Object types allow the full flexibility of capabilities to apply. In particular, for an object type
`T`, the type `id T` is a reference which can always be freely shared between threads, does not
prevent recovery of isolation or freezing, and which can cross the boundary of isolated object
subgraphs.

New object types are declared primarily with class and object declarations. A class declaration is a
sort of template for instances of the class. While an object declaration declares a unique instance
for the type. Each trait declaration also creates an object type. See [Traits](#traits) for the
nuances of traits with regards to object, value, and struct types.

### Other Object Types

In addition to object types from class and trait declarations, the following types are object types:

* Function Types
* Intersections of Object Types
* Unions of Object Types
* The special types `Any`, `Type`, and `Metatype`

## Value Types

Value types have instances that are allocated directly on the stack, in object fields, etc. This is
often shortened to simply saying that they are stack allocated. However that is not technically
correct since a value type in an object field is allocated on the heap as part of an object
instance. Additionally, there are times when a value type that would normally be on the stack must
be moved to the heap. For example, local variables can be moved to the heap when they are captured
as part of a closure. It is perhaps most accurate to say value types are allocated inline since for
any value typed variables, parameters, fields, etc. the instance is stored directly. Value types are
passed by value meaning that when passing the instance to a function or assigning an instance from
one variable to another, the instance is copied so that there are now two copies of the value, one
inline in each place.

Value types also allow the full flexibility of capabilities to apply. While they are not themselves
object types, they can contain object references to which the capabilities apply. The capabilities
on value types act exactly as if the value was an object type. For example, an `iso Value` will
become an `own Value` or `mut Value` when a copy is made the same way an isolated object would when
an alias of the object was made.

New value types are declared with value and unit value declarations. A value declaration is a sort
of template for constructing instances of a given type. Since values are copied, they limit their
fields to be `let` bindings only (`var` bindings are not allowed). Additionally, they cannot contain
fields that are isolated or owned since those could not be properly copied without introducing the
risk of divergence between the copies. Thus all fields must have an `alisable` reference capability.

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
* Tuple types
* Pointer types (i.e. `@T`)

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
compiler will ensure that owned hybrid type values have isolation recovered before the value goes
out of scope. This ensures that no hybrid type reference can outlive the value it references on the
stack since that reference would prevent recovery of isolation (except for `id` and `const` which
are discussed next).

Because `id` references can exist to isolated values, it is possible for `id` reference to structs
to reference structs that have gone out of scope. To make this safe, no access through an `id`
struct reference is allowed. They can only be compared with `@==` and `@=/=`.

The `const` capability poses problems for hybrid types because their sharing is not tracked. It
might be thought that `const` could simply be disallowed for hybrid types. However, since `const` is
transitive and a hybrid type could be a field in an object, it is always possible that the object
becomes `const` and the field is therefore `const`. So, a `const` struct instance is stored inline
and copied by value. However, to prevent a situation where a read reference exists to a `const`
struct that goes out of scope, `const` structs cannot be stored on the stack and a struct on the
stack cannot be frozen.

**TODO:** it might be possible to outlaw `id H` since combining capabilities doesn't produce it.
However, that might cause problems for generics where something like `shareable T` could result in
`id T`.

When a hybrid type is moved, the previous variable binding type changes to `void` and reports errors
for "use of moved value". This is the case because it cannot change to `id` because such a binding
does not have the same size as the struct instance.

New hybrid types are declared with struct declarations. A struct declaration is a sort of template
for constructing instances of the struct. There are not other hybrid types than those created with
struct declarations.

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
However, when a trait is used as a type for a variable, parameter, field, etc. then it is an object
type. Only object type values that implement the trait can be passed. This is because all references
are the same size and carry information allowing them to be used polymorphically. Value types and
hybrid types vary in size and do not carry the information necessary for polymorphism. Likewise, an
instance of a value type or hybrid type that implements a trait cannot be assigned into a variable
of that trait. In languages like C# and Java assigning value types into reference variables is
allowed by an implicit boxing conversion. Azoth does not do implicit boxing. This avoids the hidden
performance costs of boxing which developers are often unaware of.

## Other Types

While almost all types fall into one of object types, value types, and hybrid types, there are two
types that do not fit into these categories. The `void` type is in some ways an unit value type.
However, since it can make parameters drop out and doesn't allow `void` type variables, parameters,
fields etc. it isn't a true value type. It can be passed as a generic argument in place of all three
object types. The `never` type is a true subtype of types in all three categories. In that regard in
can be seen as being in all three categories. However, its unique behavior and meaning puts it in a
category of its own.
