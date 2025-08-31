# Generics

## Capability Constraints

The capability of a generic type parameter can be constrained by prefixing the parameter with a
capability or capability set. If one is omitted, the default is `aliasable`.

## Independence

A type parameter can be independent. This means that accessing it from `self` does not apply the
capability of `self` to it. Effectively, the declarer of the containing type "owns" that value. The
default is not independent. Simple independence is `ind`. Additionally, there is `shareable ind`
which is a restricted form of independence.

**TODO:** fully and carefully document shareable independence. `shareable independent T` is a
restriction on an independent type parameter which requires that changes in the capability do not
modify the capability of of `shareable T`. This allows the dictionary to have a field of type
`Hash_Equivalence[shareable Key]` even though `Key` is independent. The current syntax is confusing
since it seems to imply that the parameter must have a shareable capability and that is not what it
does. A better syntax is `independent(shareable) T`.

## Variance

The variance of a type parameter is controlled by suffixing it with `in`, `out`, or `nonwriteable
out`. The default is invariant. The `nonwriteable out` variance indicates that this parameter is
`out` only when the containing type in non-writeable. This allows for subtyping when something
becomes read-only. For example, while `mut List[Dog]` is not a subtype of `mut List[Animal]`, it is
true that `const List[Dog] <: const List[Animal]`.

## Default Constraints

By default, a generic type parameter does not allow `move` types.

## Allowing Move Types

A constraint like `where T: move` would imply that `T` *must* be a move type. Instead, the keyword
is placed before the type name (e.g. `where move T`). This indicates that the type variable has the
`move` limitations applied to it. Since it has those limitations enforced, it can then accept types
that require those limitations. However, it does not require that the type be that.

## Constraints

**TODO:** a better syntax for reference constraint than `T: trait`. All classes also define traits
so `trait` makes sense. However, function references are also reference types.

```azoth
where T: class  // T must be a class type declared with class or trait
where T: value  // T must be a value type
where T: struct // T must be a struct type (and hence is also a move type)
where T: move   // T must be a move type

// Unsure about these
where T: reference // T must be a reference type including classes, functions, struct reference etc.
where T: copy // T must be a copy type (i.e. value or iso struct)

where T <: S // subtype
```
