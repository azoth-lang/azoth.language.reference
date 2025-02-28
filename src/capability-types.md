# Capability Types

Capability types restrict what can be done with a value through this or other references that can
reach that value. There are five core capabilities and two temporary ones. All value and reference
types must have a capability applied to them to be full types that can be used as the types of
variables, parameters, and fields.

```grammar
capability_type
    : capability value_type
    | capability object_type
    ;
```

## Capabilities

Each of the capabilities restricts what can be done to the value either through this reference or
other references. Most capabilities are indicated by a three letter abbreviation. However, the
default capability of read-only is indicated by no capability keyword in most instances (in some
other places it is indicated by the keyword `read`).

```grammar
capability
    : "iso"
    | "mut"
    | // read (default/no capability stated)
    | "const"
    | "id"
    | "temp" "iso"
    | "temp" "const"
    ;
```

The base capabilities are:

* Isolated (`iso T`): indicates that no other readable or writeable references exist into the object
  subgraph referenced by this reference. It provides both read and write capabilities.
* Mutable (`mut T`): provides write and read access to the referenced object. Allows other writeable
  or readable references to exist to the same object.
* Read-Only (`T`): provides read access to the referenced object. Allows other writeable or readable
  references to exist to the same object. *Remember this does not mean the object cannot be mutated.
  It just can't be mutated via this reference.*
* Constant (`const T`): indicates that no writeable references exist into the object subgraph
  referenced by this reference. Thus the objects reachable via this reference are immutable and will
  never change.
* Identity (`id T`): provides very limited access but places no restrictions on what other kinds of
  references there could be to the object. Specifically, it allows references to be compared for
  reference equality and access to `let` bound `const` fields of the object.

These capabilities form their own type hierarchy. When determining whether one type is a subtype of
the other, the bare types must be subtypes but so must the capabilities. The capabilities have two
paths through the subtype relationships, namely `iso` <: `mut` <: `read` <: `id` and `iso` <:
`const` <: `read`.

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
is used, a second temporary alias is created and the variable is no longer `iso`. Typically it is
now `mut`. At key moments isolation can be recovered if the compiler can verify that no references
that could violate isolation could exist. This typically occurs when returning an `iso` type and
with the `move` expression. However, an implicit `move` can also occur on the `self` parameter when
calling a method.

When a value is moved, what is left behind is a value with the `id` capability.

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
capability viewpoint types can become indispensable. For capability viewpoint types, the read-only
capability is expressed by the `read` keyword to avoid ambiguity and confusion.

The second form of viewpoint type uses `self` as the source of the capability. This form
of capability viewpoint type can become necessary when `self` is constrained to a capability set
rather than to a specific capability. To express the return type of such methods, it is often
necessary to use a self capability viewpoint type since the capability returned depends on the
capability of `self`.

```grammar
viewpoint_type
    : explicit_capability "|>" type
    | "self" "|>" type
    ;

explicit_capability
    : "iso"
    | "mut"
    | "read"
    | "const"
    | "id"
    | "temp" "iso"
    | "temp" "const"
    ;
```

**TODO:** Perhaps this should be `explicit_capability "." type` (e.g. `const.T`) and `"self" "."
type` (e.g. `self.Foo`) instead to reflect that the types are the result of member access? The main
concerns are whether those would be obvious enough and whether there would be some ambiguity,
particularly with the `self.` form. If `self |> T` can occur in an expression then `self.T` would be
confusing and ambiguous with a member access. However, associated members are accessed from bare
types that don't have capabilities so that would seem to not be a problem.

## Capability Constraint Types

Most types have a single capability. However, the type of `self` can sometimes be given a set of
capabilities that it could have. It isn't possible to directly write such types, but the expression
`self` has one of the these capability constraint types if the `self` parameter is bounded by a
capability set.
