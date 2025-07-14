## `never` Type

```grammar
never_type
    : "never"
    ;
```

The `never` type is the bottom type in Azoth. That is, it is a subtype of all other types. A
function returning `never` or an expression of type `never` can't return a value. Instead it must
either throw an exception, cause program abandonment, or never terminate.

The never type is useful in a number of circumstances. First, it is useful for functions and
expressions which are known to never return. It is the type of various expressions like `return`,
`break`, `next`, and `throw` which don't evaluate to a value. This allows for their use in boolean
or coalescing expressions.

The `never` type can be useful in cases where a type is expected but can't occur. For example, a
function expecting a "`Result[T, Error]`" type which is either a value of type "`T`" or an error of
type "`Error`", could be passed a value of type "`Result[T, never]`" if an error is not possible in
this circumstance. The literal value "`none`" used with [optional types](optional-types.md) has the
type "`never?`".

Finally, `never` can be useful as the subtype of all types. For example, an immutable empty list can
have the type `List[never]` so that it is a subtype of all other list types.
