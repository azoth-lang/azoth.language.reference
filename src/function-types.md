# Function Types

Function types are the parameter and return types of a function. Note that function types are
contravariant in their parameter types and covariant in their return types.

```grammar
function_type
    : "(" type_list ")" "->" type
    ;

type_list
    : type{",", 0, *}
```

**TODO:** function types may need something to indicate how they handle their closure/state.
Currently thinking that `const () -> void` means the closure is constant, `mut () -> void` means it
is mutable (and thus can't be passed between threads), `() -> void` means the closure is read only
(and thus has no side effect) and `move () -> void)` means that it contains `move` types and is
consumed. The latter also makes the function type itself a move type. Thus move functions can only
be called once. One question is whether there needs to be a distinction between `mut` functions and
read only functions.

## Function Type Subtyping

Function types are contravariant in their parameter types and covariant in their return types. That
is if F1 <: F2 then for each parameter Px, F2.Px <: F1.Px. And F1.Return <: F2.Return.

The `lent` attribute of the parameters and returns also comes into play. If you translate `lent`
into which parameters can be referenced by the return value, then for a function type to be a
subtype of another it must be as or more restrictive in what parameters can be referenced by the
return value. This can get very confusing when flipping whether the return type is `lent`.
