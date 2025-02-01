# Generics

## Capability Constraints

The capability of a generic type parameter can be constrained by prefixing the parameter with a
capability or capability set. If one is omitted, the default is `aliasable`.

## Independence

A type parameter can be independent. This means that accessing it from `self` does not apply the
capability of `self` to it. Effectively, the declarer of the containing type "owns" that value. The
default is not independent. Simple independence is `ind`. Additionally, there is `sharable ind`
which is a restricted form of independence.

**TODO:** fully and carefully document shareable independence

## Variance

The variance of a type parameter is controlled by suffixing it with `in`, `out`, or `nonwriteable
out`. The default is invariant. The `nonwriteable out` variance indicates that this parameter is
`out` only when the containing type in non-writeable. This allows for subtyping when something
becomes read-only. For example, while `mut List[Dog]` is not a subtype of `mut List[Animal]`, it is
true that `const List[Dog] <: const List[Animal]`.

## Default Constraints

By default, a generic type parameter does not allow `move` types and `ref` types.

## Allowing Move and Variable Reference Types

A constraint like `where T: move` would imply that `T` *must* be a move type. Likewise, `where T:
ref` implies that `T` must be a ref type. Indeed, `where T: ref struct` is the syntax for declaring
that a type parameter must be a ref struct type.

Instead, the keyword is placed before the type name (e.g. `where move T` or `where ref T`). This
indicates that the type variable has the `move` or `ref` limitations applied to it. Since it has
those limitations enforced, it can then accept types that require those limitations. However, it
does not require that the type be that.

## Constraints

**TODO:** a better syntax for reference constraint than `T: class`

```azoth
where T: class // Then `ref T` becomes invalid
where T: struct
where T: move // T must be a move type
where T: ref struct

where T <: S // subtype
```
