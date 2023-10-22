# Constructors

Constructors are class members that provide the actions for initializing instances of a class. They
are declared using constructor declarations.

Constructors may be either named or unnamed. An unnamed constructor is identified only by the class
it constructs and the types of its parameters. A named constructor also has a name, similar to a
method name. Just like methods, named constructors may be overloaded based on the number and type of
parameters.

```grammar
constructor_declaration
    : attributes? constructor_signature constructor_body
    ;
```

## Introduction & Examples

An empty, unnamed constructor definition is equivalent to the default constructor.

```azoth
public class Example
{
    public new(mut self)
    {
    }
}
```

Class instances are created and their constructors called using the `new` keyword.

```azoth
let x = new Example();
```

A named constructor is declared by following the `new` keyword with a constructor name. When
constructing an instance, a named constructor can be called with a syntax similar to calling an
associated function, except prefixed by the `new` keyword.

```azoth
public class Example
{
    public new named(mut self)
    {
    }
}

let x = new Example.named();
```

Constructors implicitly have a mutable self parameter. The `self` parameter has special restrictions
on its use to avoid leaking of uninitialized references out of constructors (`init`?).

Constructors can optionally be given a return type. This type must be a base type or must have at
least one type parameter specified. That is, the return type may not be the type of the class the
constructor is declared in, nor a subtype or unrelated type.

```azoth
public class Example: Base_Example
{
    public new(mut self) -> Base_Example
    {
    }
}
```

It is illegal to simply repeat the class name or to use a subtype or unrelated type.

## Self Parameter

Constructors like methods have a self parameter. This is typically `mut self` regardless of the
return reference capability or whether the class is a `const` class. This is because typically
mutation is needed during the construction of the object. However, for types that are used as both
mutable and constant, it is sometimes necessary to have constructors that take a read only reference
to self. The reason for this is that when constructing from constant data, it must be possible to
store references to constant data into fields of the instance that would be mutable if the
constructor were `mut self`.

The constructor `self` parameter has special reference capabilities. As described in the sections
below, it doesn't allow passing a non-fully initialized `self` to another method or function.
Additionally, a read only self parameter still treats var fields as assignable during the first
phase of construction (the not fully initialized section).

The reference capability of the return type of a constructor is constrained by the reference
capability of the self parameter. Thus a read only self constructor cannot return a mutable
reference.

## Default Constructors

If a class has no constructor declarations, then a default constructor is created for it. This
constructor is a `published` unnamed constructor that takes no parameters. If the class has a base
class, the default constructor will call the base class constructor that is unnamed with no
parameters. If no such base class constructor exists, it is an error. No further field
initialization is done beyond running the field initializers. If this would leave a field
unassigned, that is an compile error.

## Calling Other Constructors

In a class with a base class, a constructor must either call a base class constructor or another
constructor on the class. Base class constructors are called using the `new` keyword as if it were a
method of the base class, i.e. `base.new(arguments...)`. Named constructors are called by following
the new keyword with the constructor name, i.e. `base.new.name(arguments...)`. Other constructors on
the class are called similarly using `self`, i.e. `self.new(arguments...)` or
`self.new.name(arguments...)`. Using the implicit self reference when calling constructors is not
legal. The `self` keyword must explicitly be use. For example `.new(arguments...)` would be an
error. It is an error for self constructor calls to form a cycle.

## Field Initialization Shorthand

Fields can be directly initialized from constructor arguments. This is done by prefixing the
parameter name with `.` and omitting the type. The type is determined by the type of the field.

```azoth
public new(mut self, .field)
{
}
```

## Definite Assignment

Just as any variable in a function must be definitely assigned before being used, any field must be
definitely assigned in a constructor before being used. All fields must be definitely assigned
before a reference to `self` can be shared outside of the constructor. This ensures that all objects
will be fully initialized before use. It also avoids the need to initialize all fields to some safe
default value before construction begins. Instead, it is safe to run construction on uninitialized
memory because it is guaranteed to be initialized before use. This is guaranteed by a definite
assignment analysis that divides the constructor body into two parts. The initial section in which
some fields may be uninitialized and the final section in which all fields, including fields of
subclasses are guaranteed to be initialized. The transition between these sections is the point at
which a base class constructor is called. If a base class constructor is explicitly called, that
calls determines the transition point. If the base class constructor is implicitly called, or there
is no base class, the transition point occurs as the first point where every field is definitely
assigned. The compiler inserts the implicit base class constructor call there.

For the purpose of definite field assignment there are two kinds of constructors. Constructors that
call another constructor of the current class are termed *secondary constructors*. Constructors that
do not call another constructor of the current class are termed *primary constructors*. If this
class subclasses another class, then its primary constructors will call a base class constructor
either explicitly or implicitly.

### Primary Constructors

When a primary constructor is called, it first executes all field initializers in textual order.
Then any fields initialized from an argument using the initialization shorthand are initialized in
argument order, left to right. The body of the constructor is then executed. The compiler divides
the body of the constructor into two sections. In the first section, the self instance is considered
to be not fully initialized. In the second section, the object has been fully initialized. If there
is an explicit call to a base class constructor, every statement after this call is the second
section. If there is no explicit base class constructor call, the compiler determines the first
statement at which every field has been definitely assigned. If this class is a subclass, it inserts
an implicit call to the base class default constructor at the point where all fields have been
definitely assigned.

**TODO:** For classes without a base class, we may need a clear way to delineate the separation
between these parts of the constructor.

In the not fully initialized section, the self reference may only be used to assign into fields or
access fields that have definitely been assigned. A reference to self may not be borrowed, shared,
or otherwise passed to another function or method. At the end of this section, if any fields with
optional types remain uninitialized, they are implicitly initialized to `none`.

In the fully initialized section, it is guaranteed that every field of this class, its base class
and any subclasses has been definitely assigned. Thus this section acts as a normal method with no
special rules which may perform any additional initialization steps. The intention is that by this
point, the instance should be initialized to a valid state. However, if it was not possible to put
the object into the desired state, the rest of the constructor can do so.

### Secondary Constructors

When a secondary constructor is called, no field initializers are run. Instead, execution begins in
the constructor body. Before the call to the primary constructor, the `self` reference may not be
used. Thus no fields can be assigned or read. This is because the primary constructor will
initialize every field, so any initializations performed would be overwritten. There can however, be
other statements before the call to the primary constructor for the purpose of preparing values and
state to call the primary constructor with. After the call to the primary constructor, all fields
have been definitely assigned and the constructor is treated as a normal method.

**TODO:** How does field initialization shorthand work with secondary constructors? Is it even
allowed? The initialization shouldn't be run before the call to the primary constructor, instead it
is like they should be passed along to the primary constructor. That may require special syntax or
something.

## Exceptions

Exceptions in constructors can be complex and easily lead to issues. For this reason constructors
are implicitly `no throw` instead of the default inferred throws. However, constructors can throw
exceptions if they are declared to throw them.

**TODO:** confirm this section is still correct.

If an exception is thrown in a constructor, destructors will not be run on the instance except as
part of stack unwinding as described below. Exceptions thrown in or before a base or self
constructor call simply unwind the stack, deleting any initialized field values. Exceptions thrown
after the completion of a base or self constructor call invoke the *base* classes destructor on this
instance at the point of unwinding past the base class call (since it is now a fully constructed
instance of the base class).

Note that the current classes destructor is never called so it is imperative that any resources be
properly released by the constructor.

If you catch an exception from a self or base constructor call. You must re-throw it, because the
current instance is not in a valid state.

## Partially Initialized References?

The definite assignment rules lead to some real problems. For example, imagine a tree node class
with left and right child node fields and a parent node reference. We must make the parent reference
optional and mutable, and initialize it after the child. Otherwise, we have a problem initializing
the tree even from in the parent constructor. The parent node needs to construct its two child nodes
so that it can initialize its fields. However, to construct them it must provide a reference to
itself. Yet, it is not completely initialized yet, so this is not safe. It might be possible for the
reference capabilities to provide a way out of this. There could be a reference capability which
does not allow for any methods or fields to be accessed but can only be stored with the promise that
it would be valid in the future. However, the rules for this would probably be very complicated.

## Constructor Inheritance

Swift uses convenience constructors to allow constructor inheritance. I might be able to do
something similar by making designated constructors take self as an argument while convenience
constructors don't, but must make a new object. Of course, I need to figure out how to make it
explicit that you must return the result of calling new on your own type.

## Copy Constructors

Copy constructors are special constructors used to make a copy of the class. They have the unique
property of being virtual on the type of object being copied, even though it is passed as a
parameter. They are declared with the special `copy` constructor name and take an argument of type
`Self` or `mut Self`.

```azoth
public new copy(mut self, value: Self)
{
    // copy fields
}
```

Construction proceeds through the inheritance hierarchy just as with normal constructors, starting
with the most concrete type of the object being copied. Any field that has an implicit copy
constructor and isn't explicitly assigned in the constructor is automatically copied as part of the
compiler generated copy.

Copy constructors are called with the special syntax

```azoth
let x = new copy(y);
```

Note that we do not have to specify the type being copied. It is the dynamic type of the parameter.

## Open Issues

* If some type parameters are specified, what does that do to the type parameters when constructing?
* Can the constructor method be generic so it can control type parameters?
* How are type parameters passed when calling a constructor given that they may be changed/reduced
  it seems they belong with the name not the type?
* How can one combine specifying type parameters with returning a base type?

Idea:

* Perhaps it is like the constructor is a static method on a non-generic version of the class that
  has all the same type parameters by default
