# Generics

## Capability Constraints

The capability of a generic type parameter can be constrained by prefixing the parameter with a
capability or capability set. If one is omitted, the default is `aliasable`.

## Independence

A type parameter can be independent. This means that accessing it from `self` does not apply the
capability of `self` to it. Effectively, the declarer of the containing type "owns" that value. The
default is not independent. Simple independence is declared by prefixing the generic parameter with
`independent`.

Independent generic parameters always allow `any` capability. However, this flexibility limits their
use in some scenarios. To support these use cases an independent generic parameter can have its
variation constrained to prevent breaking constrained types. The parameter is still allowed to start
with any capability. This just restricts how that capability can be changed by flow typing. For
example, the key type of a dictionary is declared `sendable independent Key`. Whatever the
capability the key is initially declared with the constrained type `sendable Key` must remain valid
for it. In the case of `sendable` this specifically restricts a `const` key type from being upcast
to an `id` key type. Just like with constrained types, it must always be possible for the capability
to be upcast to the capability set, so only capability sets containing `id` can be used to constrain
an independent parameter. It is important to remember that this constraint is *not* a constraint on
which capabilities can be used like a regular generic parameter constraint.

**TODO:** The current syntax is confusing since it seems to imply that the parameter must have a
shareable capability and that is not what it does. A alternate syntax is `independent(shareable) T`.

## Variance

The variance of a type parameter is controlled by suffixing it with `in`, `out`, or `readonly out`.
The default is invariant. The `readonly out` variance indicates that this parameter is `out` only
when the containing type in `readonly`. This allows for subtyping when something becomes read-only.
For example, while `mut List[Dog]` is not a subtype of `mut List[Animal]`, it is true that `const
List[Dog] <: const List[Animal]`.

## Default Constraints

By default, a generic type parameter does not allow `drop` types.

**TODO:** do they allow structs by default? That would also cause problems developing generic types.

## Allowing Drop Types

A constraint like `where T: drop` would imply that `T` *must* be a drop type. Instead, the keyword
is placed before the type name (e.g. `where drop T`). This indicates that the type variable has the
`drop` limitations applied to it. Since it has those limitations enforced, it can then accept types
that require those limitations. However, it does not require that the type be that.

## Constraints

**TODO:** a better syntax for reference constraint than `T: trait`. All classes also define traits
so `trait` makes sense. However, function references are also reference types.

```azoth
where T: class  // T must be a class type declared with class or trait
where T: value  // T must be a value type
where T: struct // T must be a struct type
where T: drop   // T must be a drop type

// Unsure about these
where T: reference // T must be a reference type including classes, functions, struct reference etc.

where T <: S // subtype
```
