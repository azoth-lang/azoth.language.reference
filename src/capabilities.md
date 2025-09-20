# Capabilities

Contents:

* [Why Capabilities](#why-capabilities)
  * [Fearless Concurrency](#fearless-concurrency)
  * [Defensive Copies](#defensive-copies)
  * [Compile-time Constants](#compile-time-constants)
  * [Iterator Invalidation](#iterator-invalidation)
  * [Resource Management](#resource-management)
* [Basic Capabilities](#basic-capabilities)
  * [Default Capabilities](#default-capabilities)
* [Advanced Capabilities](#advanced-capabilities)
  * [Isolated Capability](#isolated-capability)
  * [Owned Capability](#owned-capability)
  * [Temporary Capabilities](#temporary-capabilities)
  * [Init Capabilities](#init-capabilities)
* [Const Types](#const-types)
* [Capability Subtyping](#capability-subtyping)
* [Flow Typing](#flow-typing)
  * [Move](#move)
  * [Freeze](#freeze)
* [Lent Parameters](#lent-parameters)
* [Capabilities and Generics](#capabilities-and-generics)
  * [Independent Generic Parameters](#independent-generic-parameters)
* [Capabilities with Associated Types and Self](#capabilities-with-associated-types-and-self)
* [Capability Sets](#capability-sets)
  * [Constraining Generic Parameters](#constraining-generic-parameters)
  * [Flexible Self Capability](#flexible-self-capability)
  * [Constraining Independent Generic Parameters](#constraining-independent-generic-parameters)
* [Composing Capabilities](#composing-capabilities)
  * [Viewpoint Types](#viewpoint-types)
  * [Constrained Types](#constrained-types)
* [Sharing Sets](#sharing-sets)

## Why Capabilities

### Fearless Concurrency

### Defensive Copies

### Compile-time Constants

**TODO:** having strong and deep `const` allows all objects to be turned into constants at compile
time.

### Iterator Invalidation

### Resource Management

**TODO:** drop types provide resource management with deterministic cleanup

## Basic Capabilities

The basic capabilities represent permissions either for the current value or for all references to a
value. They are:

* `mut`: mutable allows both read and modification
* `read`: the default capability represented without a keyword in most cases allows reading from a
  value but not modifying it.
* `id`: the identity capability allows comparing for object identity (i.e. with `@==`, `@=/=`, and
  `.identity_hash()`) and accessing, `let` fields with `const` types.
* `const`: indicates that this value is read only by all references to it.

**TODO:** describe all the reference capabilities except `iso`, `temp`, and `init`

### Default Capabilities

The default capability of a type depends on whether the type is declared `const`. Types declared
`const` default to `const` capability. All other types default to `read` capability.

## Advanced Capabilities

### Isolated Capability

An `iso` capability indicated an isolated reference or value. This means that this is the only value
that can access the subgraph of objects reachable from this value. There is no other reference into
this subgraph. Note that objects in the subgraph can contain `const` and `id` references out to
other values. Those do not break the isolation boundary.

Isolation is a transient property. A variable or field with an isolated capability will start
isolated, however the moment that it is accessed, a temporary alias will be created and the variable
or field will no longer be isolated. Flow typing means that from that point forward in the code, the
variable or field effectively has the capability `own` or `mut` if the type is not capable of being
`own`. This means that all isolated values effectively allow mutation.

Since isolation is a transient property, in order to pass an isolated value, one must either have an
isolated temporary, or must recover isolation. Consider a function `fn take(e: iso Example)`. Given
another function `fn produce() -> iso Example`, then `take(produce)` is valid because the temporary
reference to `Example` is `iso` and is consumed by passing it to `take()`. Here no special operation
is needed. However, if one declares a variable `let v: iso Example = ...` then calling `take()` on
`v` will pose a problem because the moment `v` is referenced, there will be two aliases to the value
and it will no longer be isolated. In order to support this, one must *recover* isolation. This is
done with the `move` keyword as `take(move v)`. The expression `move v` will recover isolation,
change the type of `v` and evaluate to an isolated value. Isolation can only be recovered if the
compiler can prove that there can be no other references to the value other than local variables
whose type can be changed to ensure isolation. The type of the local variable changes depending on
what kind of type it is. If `Example` is a reference or value type, it changes to `id Example`. If
`Example` is a struct and the variable has the capability `iso` or `own` then the variable type
changes to `void` and accessing it produces an error for a "use of a moved value".

The move operation can be implicit when it is necessary on the self parameter of a method.
Otherwise, it must always be explicit.

### Owned Capability

Some types support an additional capability. Drop types and struct types support the owned
capability. The `own` capability indicates that this value is no longer isolated, but the variable
binding still has ownership of the value. For drop types, this will be used to ensure that the value
is dropped before it goes out of scope. For struct types, this is used to ensure that references to
the struct do not outlive the binding where the struct value is stored. For types that support the
owned capability, an isolated reference is promoted to `own` when an alias is made.

Changing which binding owns a value is done with the `move` keyword. Moving a struct value requires
recovering isolation just like `move` does for types that cannot be owned. For reference types which
can be owned, it is possible to pass ownership even though isolation cannot be recovered. In this
case, the two operations are distinguished by specifying what capability is being moved (e.g. `move
iso v` or `move own v`). In that case, the reference left behind remains `mut`.

**TODO:** it would be good if the move type could be inferred. If that is possible, specify the
rules and remove `move iso` vs `move own`.

### Temporary Capabilities

In additional to the above capabilities, there are two temporary capabilities `temp iso` and `temp
const`. These capabilities indicate that so long as this reference or value exists, it will be
isolated or constant. However, there are other references that have been masked off that will come
back into force when this capability goes away. For example, a `temp const` reference means that
while there may be `mut` references to the value in existence, the compiler will ensure none can be
accessed so long as the `temp const` reference exists.

### Init Capabilities

There are two special init capabilities. They are init `mut` and init read. Neither can be
explicitly declared in Azoth. The self parameter of initializers has one of these capabilities
depending on whether it is declared `mut` or read. The `self` parameter has this capability until
the call to the base initializer completes. The init capability allows for initializing fields but
it also prevents the value from being passed to other methods before it is fully initialized.

## Const Types

Types can be declared `const`. All the instances of a `const` type are constant except for the
`self` parameter of the initializer which can be init `mut` or init read. The default capability for
such types is `const`.

## Capability Subtyping

Capabilities have subtyping relationships between them. Thus a type is a subtype of another type if
the bare types are in a subtype relationship and the capabilities are subtyped. The subtype paths
are:

* `iso <: own <: mut <: read <: id`
* `iso <: const <: read`
* `const <: temp const <: read`
* `iso <: temp iso <: mut`

Note that `temp iso` is not a subtype of `own`.

## Flow Typing

In some instances, the capability of a variable or field binding is changed according to the flow of
execution. Anytime an `iso` value is accessed, it is no longer isolated and is promoted to either
`own` or `mut` if the type does not support `own`. Flow typing is also affected by `move` and
`freeze` which recover capabilities.

### Move

As described in the section about `iso`. Moving a value with `move` will recover isolation. For
reference and value types, this changes the flow typing of any variables referencing the value to
`id`. For an `own` struct type, it changes the flow type to `void` and accessing the variable will
produce an error about use of a moved value. `move` can also be used to move ownership only. In that
case, the variables left behind remain `mut`.

### Freeze

In addition to recovering isolation with `move`, one can recover to `const` with `freeze`. This is
similar to recovering isolation except that it allows read references to exist while recovering and
leaves the type of an variables referencing the value as `read`.

## Lent Parameters

Parameters to functions and methods can be declared `lent`. A lent parameter indicates that when the
function returns, the object subgraph reachable from that parameter will remain disconnected from
the object subgraph of any other parameter. This allows one to manipulate mutable parameters while
not causing the compiler to prevent moving or freezing because there might have been an additional
reference introduced. Since `const` and `id` references do not break isolation, a `lent` parameter
cannot be `const` or `id`.

## Capabilities and Generics

Generic arguments include a capability. Consider the trait `Iterator[any T]`. When specifying a type
to iterate over, one includes the capability. For example, one could have a `mut Iterator[mut
Example]` or a `mut Iterator[const Example]`. Thus within the iterator type, the type `T` already
has a capability and cannot have a capability specified (e.g. `const T` would be invalid).

There are more complexities when working with generic types and capabilities as described in the
section on generics.

### Independent Generic Parameters

Generic parameters can be `independent`. This means that the capability of that parameter can change
according to the flow typing rules. Effectively, it creates layers within generic types where the
capability acts as if it was declared as a binding alongside the containing generic type. This
allows greater flexibility for collections like `List[independent T]`. Independent generic types
have a number of unique behaviors. For example, they are effectively `lent` when used as parameters.
Additionally, they cannot be combined with viewpoint types as described in the next section. A full
discussion of them can be found in the section on generics.

## Capabilities with Associated Types and Self

Associated types declare types which, unlike generic parameter types, do not have a capability. Thus
for an associated type `A` it must still be used with capabilities (e.g. `mut A`). Likewise, the
`Self` type is a sort of implicit associated type and must have a capability specified with it.

## Capability Sets

Capabilities are grouped into overlapping capability sets which are used to restrict which
capabilities can occur in certain places. The capability sets are:

* `readable`: able to be read from. (`own`, `mut`, `const`, `temp const`, `read`)
* `shareable`: able to be safely shared between threads. (`const`, `id`)
* `aliasable`: able to be aliased by a value with the same capability. (`mut`, `const`, `temp const`, `read`, `id`)
* `sendable`: able to be sent between threads. (`iso`, `const`, `id`)
* `readonly`: able to be read from but not able to mutate. (`const`, `temp const`, `read`, `id`)
* `temporary`: a temporary capability (`temp iso`, `temp const`)
* `any`: any capability (`iso`, `temp iso`, `own`, `mut`, `const`, `temp const`, `read`, `id`)

### Constraining Generic Parameters

Generic parameters can have their allows capabilities restricted by specifying a capability set or a
specific capability before the generic parameter (e.g. `Example[readable T]`). Only the given
capabilities can be used when specifying that generic argument. If no constraint is specified, the
default constraint is `aliasable`.

### Flexible Self Capability

While a regular parameter must have a specific capability. The `self` parameter can instead be
constrained to a set of capabilities. This allows a single method to operate on instances with a
range of capabilities. For example, one can declare a method with `readable self` so that it can
operate on any capability that can be read from.

### Constraining Independent Generic Parameters

Independent generic parameters always allow `any` capability. However, this flexibility limits their
use in some scenarios. To support these use cases an independent generic parameter can have its
variation constrained to prevent breaking constrained types (see [Constrained
Types](#constrained-types)). The parameter is still allowed to start with any capability. This just
restricts how that capability can be changed by flow typing. For example, the key type of a
dictionary is declared `sendable independent Key`. Whatever the capability the key is initially
declared with the constrained type `sendable Key` must remain valid for it. In the case of
`sendable` this specifically restricts a `const` key type from being upcast to an `id` key type.
Just like with constrained types, it must always be possible for the capability to be upcast to the
capability set, so only capability sets containing `id` can be used to constrain an independent
parameter. It is important to remember that this constraint is *not* a constraint on which
capabilities can be used like a regular generic parameter constraint.

## Composing Capabilities

### Viewpoint Types

When working with generic types, it is sometimes necessary to express how a generic type would be
changed by access from a certain capability or from `self`. This is done with viewpoint types.

For a generic type `T`, the type *`<capability>`*`.T` gives the type that results from accessing a
field of type `T` with whatever capability it has from an instance with the given capability. For
example `const.T` would result in either a `const` type or `id` type.

For a generic type `T`, the type `self.T` gives the type that results from accessing a field of type
`T` with whatever capability it has from the self instance with whatever capability it has. Methods
with a specific capability for `self` do not allow this. Only methods with `self` constrained to a
capability set allow such types.

### Constrained Types

When working with generic types, it is sometime necessary to express the concept that a generic type
has been cast up to a capability in a set. This is done by prefixing the generic type with the
capability set, i.e. *`<capability-set>`*` T`. This gives a new type whose capability is in the
given set. Since it must always be possible to do this, only capability sets that contain `id` can
be used in this way.

## Sharing Sets

Capabilities are enforced by the compiler. Permission capabilities are easy to enforce since they
straightforwardly restrict what can be done with a value. Advanced capabilities are more difficult
to enforce since they represent properties of the entire set of references and the object graph. To
enforce this the compiler must track how references may relate to each other. For example, it must
be able to determine when two references may refer to the same object or one reference may refer to
an object reachable from another. When doing this, the compiler must be conservative. There will
necessarily be code which is logically safe but which the compiler cannot determine is safe. The
compiler's analysis is based on sharing set.

A sharing set, is a set of values that may in some way alias each other or where one value may be
reachable from the other. Consider the function `fn example(f: mut Foo) -> mut Bar`. When calling
this function `example(foo)` the compiler must assume that it is possible that either the `Foo`
instance references the `Bar` instance or vice versa. It therefore places those two references in
the same sharing set. It would then be impossible to move or freeze either of those values until the
other goes out of scope since as long as the other is in scope, it is possible that it violates the
isolation of the other. If on the other hand, `example` where declared with a `lent` parameter then
the compiler could assume that the two instances did not reference each other and would not have to
place them in the same sharing set. I full description of this analysis would be too complicated to
give here, but this gives a basic idea of how the analysis works.
