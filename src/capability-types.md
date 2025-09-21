# Capability Types

Capability types restrict what can be done with a value through this or other references that can
reach that value. There are six core capabilities and two temporary ones. All value and reference
types must have a capability applied to them to be full types that can be used as the types of
variables, parameters, and fields.

```grammar
capability_type
    : capability value_type
    | capability object_type
    | capability struct_type
    ;
```

Contents:

* [Capabilities](#capabilities)
  * [Temporary Capabilities](#temporary-capabilities)
  * [Flow Typing](#flow-typing)
* [Viewpoint Types](#viewpoint-types)
* [Capability Constraint Types](#capability-constraint-types)
* [Constrained Types](#constrained-types)

## Capabilities

Each of the capabilities restricts what can be done to the value either through this reference or
other references. Most capabilities are indicated by a three letter abbreviation. However, the read
capability is declared via the default capability indicated by no capability keyword (in some other
places it is indicated by the keyword `read`).

```grammar
capability
    : "iso"
    | "own"
    | "mut"
    | ε // default (either read or const depending on the type it is applied to)
    | "const"
    | "id"
    | "temp" "iso"
    | "temp" "const"
    ;
```

The base capabilities are:

* Isolated (`iso T`): indicates that no other readable or writeable references exist into the object
  subgraph referenced by this value. It provides both read and write capabilities.
* Owned (`own T`): is only valid for a hybrid type or a drop type. Indicates that this binding owns
  the value and provides write and read access to the value. Allows other writeable or readable
  references to exist to the object subgraph. There can be only one `own` reference to a value. The
  `own` references must from a chain from the stack.
* Mutable (`mut T`): provides write and read access to the value. Allows other writeable or readable
  references to exist to the object subgraph.
* Read-Only (`T`): provides read access to the referenced value. Allows other writeable or readable
  references to exist to the object subgraph. *Remember this does not mean the object cannot be
  mutated. It just can't be mutated via this reference.*
* Constant (`const T`): indicates that no writeable references exist into the object subgraph
  referenced by this value. Thus the objects reachable via this value are immutable and will never
  change.
* Identity (`id T`): provides very limited access but places no restrictions on what other kinds of
  references there could be to the object. Specifically, it allows references to be compared for
  reference equality and access to `let` bound `const` fields of the object.

These capabilities form their own type hierarchy. When determining whether one type is a subtype of
the other, the bare types must be subtypes but so must the capabilities. The capabilities have two
paths through the subtype relationships, namely `iso` <: `own` <: `mut` <: `read` <: `id` and `iso`
<: `const` <: `read`.

### Temporary Capabilities

Two capabilities can be made temporary, namely `temp iso` and `temp const`. When passing a value
without that capability to a method expecting a temporary capability, the compiler will enforce that
the temporary capability applies as long as the reference with that capability could exist. While
`temp iso` <: `temp const`, other subtyping relationships hold, namely `iso` <: `temp iso` <: `mut`
and `const` <: `temp const` <: `read`.

### Flow Typing

While the regular type of a variable cannot change, the capability of a type can change along the
control flow of the code. This is called flow typing. Specifically, the `iso` and `temp iso`
capabilities cannot be directly used for member access. Whenever a variable with those capabilities
is used, a second temporary alias is created and the variable is no longer `iso`. It is now either `own` or `mut` for types that do not support `own`. At key moments isolation can be recovered if the compiler can verify that no references
that could violate isolation could exist. This typically occurs when returning an `iso` type and
with the `move` expression. However, an implicit `move` can also occur on the `self` parameter when
calling a method.

When a reference type or value type value is moved, what is left behind is a value with the `id`
capability. When an `own` hybrid type is moved, what is left behind has the type `void` and reports
an error for "use of moved value" if used.

Another way that capabilities can change is via `freeze`. After a value has been frozen, all
references to it and objects reachable from it will be `const`.

## Viewpoint Types

Sometimes, it is necessary to express the type that would result from a member access. This is what
viewpoint types are for. They produce a new type with the capability that would result from
accessing a member of a given type from an object with a given capability. The source of the
capability is listed first, followed by an open triangle arrow and then the type of the member.

The first form of of viewpoint type is the basic capability viewpoint type. This produces the type
that would result when accessing a member from a value with a standard capability. If the member
type is not a generic parameter, then it is usually not necessary to use a basic capability
viewpoint type because one knows exactly the capability that would result. But if the type is a
generic parameter or contains a generic parameter that would be affected by the type, then
capability viewpoint types can become indispensable. For capability viewpoint types, the read
capability is expressed by the `read` keyword to avoid ambiguity and confusion.

The second form of viewpoint type uses `self` as the source of the capability. This form of
capability viewpoint type can become necessary when `self` is constrained to a capability set rather
than to a specific capability. To express the return type of such methods, it is often necessary to
use a self capability viewpoint type since the capability returned depends on the capability of
`self`.

```grammar
viewpoint_type
    : explicit_capability "." type
    | "self" "." type
    ;

explicit_capability
    : "iso"
    | "own"
    | "mut"
    | "read"
    | "const"
    | "id"
    | "temp" "iso"
    | "temp" "const"
    ;
```

## Capability Constraint Types

Most types have a single capability. However, the type of `self` can sometimes be given a set of
capabilities that it could have. It isn't possible to directly write such types, but the expression
`self` has one of the these capability constraint types if the `self` parameter is bounded by a
capability set.

## Constrained Types

For generic types is is sometime necessary to upcast them to a capability within a capability set.
This can be done by prefixing the generic parameter type with the capability set (e.g. `sendable
T`). Since this must always be valid, this is allowed only when the capability contains `id`.
