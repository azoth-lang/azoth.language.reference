# Initializers

Initializers are class and struct members that provide the actions for initializing instances. They
are declared using initializer declarations.

Initializers may be either named or unnamed. An unnamed initializer is identified only by the type
it constructs and the types of its parameters. A named initializer also has a name, similar to a
method name. Just like methods, named initializers may be overloaded based on the number and type of
parameters.

```grammar
initializer_declaration
    : attributes? initializer_signature initializer_body
    ;
```

## Introduction & Examples

An empty, unnamed initializer definition is equivalent to the default initializer.

```azoth
public class Example
{
    public init(mut self)
    {
    }
}
```

Type instances are created and their initializers called using the type name as if it were a
function.

```azoth
let x = Example();
```

A named initializer is declared by following the `init` keyword with an initializer name. When
initializing an instance, a named initializer can be called with a syntax like calling an associated
function.

```azoth
public class Example
{
    public init named(mut self)
    {
    }
}

let x = Example.named();
```

## Return Type

By default initializers return the type `mut Self` for non-const types and `const Self` for const
types.

Initializers can optionally be given a return type. This type must be a base type or must have at
least one type parameter specified. That is, the return type may not be the type of the class or
struct the initializer is declared in, nor a subtype or unrelated type.

```azoth
public class Example: Base_Example
{
    public init(mut self) -> Base_Example
    {
    }
}
```

It is illegal to simply repeat the class or struct name or to use a subtype or unrelated type.

## Self Parameter

Initializers, like methods, have a self parameter. For classes, this is typically `mut self`
regardless of the return reference capability or whether the class is a `const` class. This is
because typically mutation is needed during the initialization of the object. However, for types
that are used as both mutable and constant, it is sometimes necessary to have initializers that take
a read only reference to self. The reason for this is that when initializing from constant data, it
must be possible to store references to constant data into fields of the instance that would be
mutable if the initializer were `mut self`.

The initializer `self` parameter has special reference capabilities. As described in the sections
below, it doesn't allow passing a non-fully initialized `self` to another method or function.
Additionally, a read only self parameter still treats `var` fields as assignable during the first
phase of construction (the not fully initialized section).

The reference capability of the return type of an initializer is constrained by the reference
capability of the self parameter. Thus a read only self initializer cannot return a mutable
reference.

## Default Initializers

If a class or struct has no initializer declarations, then a default initializer is created for it.
This initializer is a `published` unnamed initializer that takes no parameters. If the class has a
base class, the default initializer will call the base class initializer that is unnamed with no
parameters. If no such base class initializer exists, it is an error. No further field
initialization is done beyond running the field initializers. If this would leave a field
unassigned, that is an compile error.

## Calling Other Initializers

In a class with a base class, an initializer must either call a base class initializer or another
initializer on the class. Base class initializers are called using the `init` keyword as if it were
a method of the base class, i.e. `base.init(arguments...)`. Named initializers are called by
following the init keyword with the initializer name, i.e. `base.init.name(arguments...)`. Other
initializers on the class are called similarly using `self`, i.e. `self.init(arguments...)` or
`self.init.name(arguments...)`. Using the implicit self reference when calling initializers is not
legal. The `self` keyword must explicitly be use. For example `.init(arguments...)` would be an
error. It is an error for self initializer calls to form a cycle.

## Field Initialization Shorthand

Fields can be directly initialized from initializer arguments. This is done by prefixing the
parameter name with `.` and omitting the type. The type is determined by the type of the field.

```azoth
public init(mut self, .field)
{
}
```

## Definite Assignment

Just as any variable in a function must be definitely assigned before being used, any field must be
definitely assigned in an initializer before being used. All fields must be definitely assigned
before a reference to `self` can be shared outside of the initializer. This ensures that all
instances will be fully initialized before use. It also avoids the need to initialize all fields to
some safe default value before initialization begins. Instead, it is safe to run initialization on
uninitialized memory because it is guaranteed to be initialized before use. This is guaranteed by a
definite assignment analysis that divides the initializer body into two parts. The initial section
in which some fields may be uninitialized and the final section in which all fields, including
fields of subclasses are guaranteed to be initialized. The transition between these sections is the
point at which a base class initializer is called. If a base class initializer is explicitly called,
that call determines the transition point. If the base class initializer is implicitly called, or
there is no base class, the transition point occurs as the first point where every field is
definitely assigned. The compiler inserts the implicit base class initializer call there.

For the purpose of definite field assignment there are two kinds of initializers. Initializers that
call another initializer of the current type are termed *secondary initializers*. Initializers that
do not call another initializer of the current type are termed *primary initializers*. If this class
subclasses another class, then its primary initializers will call a base class initializer either
explicitly or implicitly.

### Primary Initializers

When a primary initializer is called, it first executes all field initializers in textual order.
Then any fields initialized from an argument using the initialization shorthand are initialized in
argument order, left to right. The body of the initializer is then executed. The compiler divides
the body of the initializer into two sections. In the first section, the self instance is considered
to be not fully initialized. In the second section, the instance has been fully initialized. If
there is an explicit call to a base class initializer, every statement after this call is the second
section. If there is no explicit base class initializer call, the compiler determines the first
statement at which every field has been definitely assigned. If this class is a subclass, it inserts
an implicit call to the base class no-arg initializer at the point where all fields have been
definitely assigned.

**TODO:** For classes without a base class (and structs), we may need a clear way to delineate the
separation between these parts of the initializer.

In the not fully initialized section, the self reference may only be used to assign into fields or
access fields that have definitely been assigned. A reference to self may not be used or passed
to another function or method. At the end of this section, if any fields with optional types remain
uninitialized, they are implicitly initialized to `none`.

In the fully initialized section, it is guaranteed that every field of this class, its base class
and any subclasses has been definitely assigned. Thus this section acts as a normal method with no
special rules which may perform any additional initialization steps. The intention is that by this
point, the instance should be initialized to a valid state. However, if it was not possible to put
the object into the desired state, the rest of the initializer can do so.

### Secondary Initializers

When a secondary initializer is called, no field initializers are run. Instead, execution begins in
the initializer body. Before the call to the primary initializer, the `self` reference may not be
used. Thus no fields can be assigned or read. This is because the primary initializer will
initialize every field, so any initializations performed would be overwritten. There can, however,
be other statements before the call to the primary initializer for the purpose of preparing values
and state to call the primary initializer with. After the call to the primary initializer, all
fields have been definitely assigned and the initializer is treated as a normal method.

**TODO:** How does field initialization shorthand work with secondary initializers? Is it even
allowed? The initialization shouldn't be run before the call to the primary initializer, instead it
is like they should be passed along to the primary initializer. That may require special syntax or
something.

## Exceptions

Exceptions in initializers can be complex and easily lead to issues. For this reason initializers
are implicitly `no throw` instead of the default inferred throws. However, initializers can throw
exceptions if they are declared to throw them.

**TODO:** confirm this section is still correct.

If an exception is thrown in a initializer, destructors will not be run on the instance except as
part of stack unwinding as described below. Exceptions thrown in or before a base or self
initializer call simply unwind the stack, deleting any initialized field values. Exceptions thrown
after the completion of a base or self initializer call invoke the *base* class's destructor on this
instance at the point of unwinding past the base class call (since it is now a fully constructed
instance of the base class).

Note that the current class's destructor is never called so it is imperative that any resources be
properly released by the initializer.

If you catch an exception from a self or base initializer call. You must re-throw it, because the
current instance is not in a valid state.

## Partially Initialized References?

The definite assignment rules lead to some real problems. For example, imagine a tree node class
with left and right child node fields and a parent node reference. We must make the parent reference
optional and mutable, and initialize it after the child. Otherwise, we have a problem initializing
the tree even from in the parent initializer. The parent node needs to instantiate its two child
nodes so that it can initialize its fields. However, to instantiate them it must provide a reference
to itself. Yet, it is not completely initialized yet, so this is not safe. It might be possible for
the reference capabilities to provide a way out of this. There could be a reference capability which
does not allow for any methods or fields to be accessed but can only be stored with the promise that
it would be valid in the future. However, the rules for this would probably be very complicated.

## Open Issues

* If some type parameters are specified, what does that do to the type parameters when initializing?
* Can the initializer method be generic so it can control type parameters?
* How are type parameters passed when calling an initializer given that they may be changed/reduced
  it seems they belong with the name not the type?
* How can one combine specifying type parameters with returning a base type?

Idea:

* Perhaps it is like the initializer is a static method on a non-generic version of the type that
  has all the same type parameters by defaults?
